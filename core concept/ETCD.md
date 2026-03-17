# ETCD

## 개요

분산형 key-value store로, Kubernetes 클러스터의 모든 상태 정보를 저장하는 데이터베이스.

- 클러스터의 노드, Pod, ConfigMap, Secret, ServiceAccount 등 모든 오브젝트 정보 저장
- 기본 포트: **2379** (클라이언트 통신), **2380** (피어 간 통신)
- `kubeadm`으로 구성 시 `kube-system` 네임스페이스에 Static Pod으로 실행됨

---

## ETCD와 kubectl의 관계

`kubectl get`, `kubectl apply` 등 모든 명령은 kube-apiserver를 통해 ETCD에 읽기/쓰기를 수행한다.

```
kubectl → kube-apiserver → etcd
```

---

## ETCD 데이터 저장 경로

| 구성 방식 | 데이터 경로 |
|---|---|
| kubeadm | `/var/lib/etcd` |
| 수동 설치 | `--data-dir` 플래그로 지정 |

---

## etcdctl 사용법

ETCD를 조작하는 CLI 도구. CKA 시험에서는 주로 **백업/복구**에 사용한다.

### API 버전 설정 (반드시 v3 사용)

```bash
export ETCDCTL_API=3
```

### 주요 명령어

```bash
# key-value 저장
etcdctl put <key> <value>

# key-value 조회
etcdctl get <key>

# 스냅샷 백업
etcdctl snapshot save <백업파일경로>

# 스냅샷 상태 확인
etcdctl snapshot status <백업파일경로>

# 스냅샷 복구
etcdctl snapshot restore <백업파일경로> --data-dir <복구경로>
```

### TLS 인증서 옵션 (kubeadm 환경)

kubeadm 구성에서는 TLS가 필수이므로 아래 옵션을 함께 사용해야 한다.

```bash
etcdctl snapshot save /opt/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

---

## 백업 및 복구 (CKA 핵심)

### 백업

```bash
export ETCDCTL_API=3

etcdctl snapshot save /opt/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### 복구

**1단계**: 스냅샷을 새 데이터 디렉토리로 복구

```bash
etcdctl snapshot restore /opt/etcd-backup.db \
  --data-dir /var/lib/etcd-restore
```

**2단계**: ETCD Static Pod manifest 수정

```bash
vi /etc/kubernetes/manifests/etcd.yaml
```

`--data-dir` 값과 `hostPath` 볼륨 경로를 복구 경로로 변경:

```yaml
volumes:
- hostPath:
    path: /var/lib/etcd-restore   # 변경
    type: DirectoryOrCreate
  name: etcd-data
```

**3단계**: ETCD Pod 재시작 확인

```bash
kubectl get pods -n kube-system
# etcd Pod이 재생성될 때까지 대기
```

---

## ETCD Static Pod manifest 위치

```
/etc/kubernetes/manifests/etcd.yaml
```

인증서 관련 주요 플래그:

```yaml
- --cert-file=/etc/kubernetes/pki/etcd/server.crt
- --key-file=/etc/kubernetes/pki/etcd/server.key
- --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
- --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
- --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
- --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
- --data-dir=/var/lib/etcd
- --listen-client-urls=https://127.0.0.1:2379
```

---

## 고가용성(HA) 구성에서의 ETCD

- HA 클러스터에서는 여러 컨트롤 플레인 노드에 ETCD가 분산 배치됨
- Raft 합의 알고리즘 사용 → **과반수(Quorum)** 노드가 살아있어야 동작
- 권장 노드 수: **홀수** (3, 5, 7...)

| 노드 수 | Quorum | 허용 장애 수 |
|---|---|---|
| 3 | 2 | 1 |
| 5 | 3 | 2 |
| 7 | 4 | 3 |

- **Stacked ETCD**: ETCD가 컨트롤 플레인 노드 위에서 실행 (kubeadm 기본값)
- **External ETCD**: ETCD가 별도 외부 노드에서 실행 (더 높은 가용성)