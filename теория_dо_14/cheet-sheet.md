export KUBECONFIG=~/.kube/do14-config

# сколько памяти всего на каждой ноде и сколько уже занято
kubectl top nodes 2>/dev/null || echo "metrics-server не установлен — глянем иначе"

echo "=== ёмкость и доступно по нодам ==="
kubectl get nodes -o custom-columns='NODE:.metadata.name,ВСЕГО_ПАМЯТИ:.status.capacity.memory,ДОСТУПНО:.status.allocatable.memory'

echo "=== сколько памяти уже назапрошено подами на каждой ноде ==="
for node in $(kubectl get nodes -o jsonpath='{.items[*].metadata.name}'); do
  echo "--- $node ---"
  kubectl describe node "$node" | grep -A5 "Allocated resources" | grep -i memory
done








