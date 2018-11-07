# ghichep-k8s
## K8s pods:
1 set các container được khởi tạo cùng nhau, mỗi container trong pod chạy 1 process và 1 port duy nhất. Có thể coi pod như 1 VM và các container trong đó là các tiến trình. Các container trong 1 pod có thể giao tiếp với nhau thông qua localhost.
Các container trong pod share nhau về network cũng như storage

## K8s Deployment
  Định nghĩa cách triển khai 1 pod bao gồm cả các thông tin về replicas, phương thức release, rollback...
  Ví dụ :
  
  ```apiVersion: extensions/v1beta1
kind: Deployment                                        
metadata:
  name: client
spec:
  replicas: 2                                  
  minReadySeconds: 15
  strategy:                           # các chiến lược release hoặc update 
    type: RollingUpdate                           
    rollingUpdate: 
      maxUnavailable: 1               # Số lượng pod tối đa có thể bị down khi triển khai           
      maxSurge: 1                     # Số lượng pod tối đa có thể thêm vào các pod đang chạy
  template:                           # Template giống như định nghĩa 1 pod      
    metadata:
      labels:
        app: client                       
    spec:
      containers:
        - image: client
          imagePullPolicy: Never            
          name: client
          ports:
            - containerPort: 80
```

#### deployment strategy:
- Recreate: Phù hợp cho triển khai khi dev. K8s sẽ terminate toàn bộ pod đang chạy và khởi tạo pod mới.
 ```spec:
  replicas: 3
  strategy:
    type: Recreate
 ```
 
- Ramped – slow rollout: hay còn gọi là rolling update. Triển khai 1 cụm replica set mới đồng thời giảm số lượng của cụm replica set cũ cho tới khi cụm mới đạt số lượng pod quy định.
```spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # how many pods we can add at a time
      maxUnavailable: 0  # maxUnavailable define how many pods can be unavailable
                         # during the rolling update
```

- Blue/Green: phương pháp này khởi tạo và chạy đồng thời 2 phiên bản của dịch vụ. Sau khi test version mới thành công thì sẽ thay đổi Service trỏ vào cụm mới dùng selector label
```apiVersion: v1
kind: Service
metadata:
 name: my-app
 labels:
   app: my-app
spec:
 type: NodePort
 ports:
 - name: http
   port: 8080
   targetPort: 8080

 # Note here that we match both the app and the version.
 # When switching traffic, we update the label “version” with
 # the appropriate value, ie: v2.0.0
 selector:
   app: my-app
   version: v1.0.0
```

- Canary: triển khai song song 2 version, service sẽ load balance giữa cả version mới và cũ. Sau 1 thời gian nếu không có lỗi thì sẽ scale up replica set mới và xóa bỏ deployment cũ.

## K8s service:
 #### Network type:
- ClusterIP: type mặc định, type này gán cho service 1 IP nội bộ trong k8s cluster. Service chỉ có thể gọi tới từ nội bộ các pod trong k8s cluster
- NodePort: map port của service tới 1 port bất kì thuộc range 30000-32767 tới tất cả các node thuộc cluster. Bên ngoài có thể gọi tới service thông qua nodeIP:nodePort
- LoadBalancer: expose service ra ngoài dùng load balancer của cloud provider. Chỉ dùng khi k8s cluster thuộc cloud provider như GKE, AKS, EKS. Service được map tới 1 public IP của cloud và có thể gọi từ bên ngoài.
- ExternalName: map service tới 1 domain name
- ExternalIPs: map service tới 1 ip public của node thuộc cluster. Có thể gọi tới từ bên ngoài thông qua publicIP:port

 #### Selector:
   Thường trong 1 service sẽ có nhiều pod chạy nhiều process trên các cổng và Ip khác nhau. Vì vậy, để cho service biết nên trỏ tới pod nào để expose ra ngoài, ta sử dụng label trên pod và selector trên service.
   Ví dụ:
   
   ```apiVersion: v1
kind: Pod
metadata:
  name: client
  labels:           # => label app = client cho tất cả các pod
    app: client
spec:
  containers:
    - image: client
      imagePullPolicy: Never
      name: client
      ports:
        - containerPort: 80
   ```
        
        
   ```apiVersion: v1
kind: Service              
metadata:
  name: client-lb
spec:
  type: LoadBalancer      
  ports:
  - port: 80               
    protocol: TCP          
    targetPort: 80         
  selector:                
    app: client      # => selector trỏ tới các pod có label app = client
   ```




