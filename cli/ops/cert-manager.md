# cert-manager ops

cert-manager Helm 설치 후 자체서명 ClusterIssuer 적용
```bash
kubectl apply -f k8s/configmap/cert-manager-clusterissuer.yaml
```

ClusterIssuer 확인
```bash
kubectl get clusterissuer selfsigned-issuer
```
