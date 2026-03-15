# CKA 추가 학습 목록

## kube-proxy
- [ ] kube-proxy 동작 모드: iptables mode vs IPVS mode 차이, 언제 IPVS가 더 유리한지
- [ ] kube-proxy가 API 서버를 watch하여 Service/Endpoint 변경을 감지하고 iptables/ipvs 규칙을 업데이트하는 흐름

## Service
- [ ] Service 종류별 동작 방식: ClusterIP / NodePort / LoadBalancer / ExternalName
- [ ] Service 종류별 트래픽 흐름 (CKA 자주 출제)
- [ ] Endpoints / EndpointSlice: Service가 실제로 트래픽을 어느 Pod로 보내는지 결정하는 객체

## Pod
- [ ] Pod Lifecycle: Pending → Running → Succeeded/Failed, Init Container 순서
- [ ] Static Pod: kubelet이 직접 관리하는 Pod, Control Plane 구성 요소가 이 방식으로 실행됨
- [ ] Multi-container 패턴: Sidecar / Ambassador / Adapter 패턴 구분
- [ ] Pod의 네트워크 네임스페이스: 같은 Pod 내 컨테이너가 왜 localhost로 통신 가능한지 (pause 컨테이너)
