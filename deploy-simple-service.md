### Create a deployment for the store-products service with four replicas
```sh
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-products
  labels:
    app: store-products
spec:
  replicas: 4
  selector:
    matchLabels:
      app: store-products
  template:
    metadata:
      labels:
        app: store-products
    spec:
      containers:
      - name: store-products
        image: linuxacademycontent/store-products:1.0.0
        ports:
        - containerPort: 80
EOF
```

### Create a store-products service and verify that you can access it from the busybox testing pod.
```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
    name: store-products-svc
spec:
    type: ClusterIP
    selector:
      app: store-products
    ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF
```
### Check result
```sh
kubectl exec busybox -- curl -s store-products-svc
{
  "Products":[
    {
            "Name":"Apple",
            "Price":1000.00,
    },
    {
            "Name":"Banana",
            "Price":5.00,
    },
    {
            "Name":"Orange",
            "Price":1.00,
    },
    {
            "Name":"Pear",
            "Price":0.50,
    }
  ]
}
```
