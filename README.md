# MESH

**MESH** adalah singkatan dari "Microservices, Events, Streams, and APIs as the new paradigm." Ini adalah konsep yang diperkenalkan oleh Gartner untuk merinci empat aspek utama yang dianggap penting dalam arsitektur mikroservis:

1. **Microservices (M):** Ini adalah pendekatan pengembangan perangkat lunak di mana sebuah aplikasi dibangun sebagai kumpulan layanan kecil, independen, dan terpisah. Setiap layanan ini berfokus pada satu fungsi bisnis tunggal dan dapat diimplementasikan, diperbarui, dan dikelola secara independen.

2. **Events (E):** MESH menekankan pentingnya manajemen peristiwa atau kejadian dalam arsitektur mikroservis. Peristiwa adalah pemberitahuan tentang perubahan status atau kejadian yang terjadi di dalam sistem. Pemanfaatan peristiwa memungkinkan sistem mikroservis berkomunikasi secara longgar dan dapat berkembang secara independen.

3. **Streams (S):** MESH memasukkan konsep aliran data untuk menangani peristiwa. Streams menciptakan cara untuk mengelola dan menganalisis data yang dihasilkan oleh peristiwa dalam waktu nyata. Ini dapat membantu dalam pemrosesan dan pengambilan keputusan cepat berdasarkan aliran peristiwa yang terjadi.

4. **APIs (A):** MESH menyoroti pentingnya antarmuka pemrograman aplikasi (API) sebagai cara untuk menghubungkan layanan dan komponen dalam arsitektur mikroservis. API memungkinkan komunikasi yang efisien antara layanan dan memfasilitasi integrasi antara bagian-bagian yang berbeda dari aplikasi.

Secara keseluruhan, MESH memberikan pandangan holistik tentang bagaimana mikroservis, peristiwa, aliran data, dan API dapat digunakan bersama-sama untuk menciptakan arsitektur yang lebih lentur, skalabel, dan mudah dikelola. Pendekatan ini dianggap sebagai landasan untuk membangun sistem yang responsif dan adaptif dalam menghadapi perubahan kebutuhan bisnis dan teknologi.


# ISTIO

**Istio** adalah sebuah platform layanan dan jaringan open-source yang dirancang untuk menghubungkan, mengamankan, mengendalikan, dan mengamati layanan mikroservis dalam suatu lingkungan komputasi terdistribusi. Istio memberikan solusi untuk beberapa tantangan yang muncul dalam pengelolaan dan berkomunikasi antar layanan dalam arsitektur mikroservis.

Beberapa fitur kunci Istio meliputi:

1. **Traffic Management (Manajemen Lalu Lintas):** Istio memungkinkan kontrol yang lebih baik atas lalu lintas antar layanan dengan memberikan kemampuan untuk merutekan lalu lintas, menerapkan pembagian beban, dan melakukan pemutusan atau penyatuan lalu lintas.

2. **Security (Keamanan):** Istio menyediakan mekanisme otentikasi, otorisasi, dan enkripsi untuk layanan mikroservis. Ini membantu melindungi komunikasi antar layanan dan memastikan hanya layanan yang sah yang dapat berkomunikasi satu sama lain.

3. **Policy Enforcement (Penegakan Kebijakan):** Istio memungkinkan definisi dan penegakan kebijakan keamanan dan kontrol akses, seperti pembatasan akses berbasis peran (RBAC) dan kontrol akses jaringan.

4. **Telemetry (Telemetri):** Istio memberikan alat untuk pengumpulan dan pemantauan data kinerja dan aktivitas sistem secara real-time. Ini termasuk pemantauan lalu lintas, jejak panggilan (tracing), dan logging.

5. **Service Mesh:** Istio berfungsi sebagai service mesh, yang berarti itu menyediakan jaringan tambahan yang melibatkan proxy sisi layanan (sidecar proxy) di sepanjang setiap kontainer layanan. Proxy ini memungkinkan Istio untuk memantau, mengamankan, dan mengontrol lalu lintas tanpa memerlukan perubahan signifikan pada kode aplikasi.

Istio sering digunakan bersama dengan orkestrasi kontainer seperti Kubernetes, tetapi dapat diintegrasikan dengan berbagai teknologi dan lingkungan. Dengan menyediakan fitur-fitur ini, Istio membantu organisasi mengelola dan memperbaiki kompleksitas yang terkait dengan arsitektur mikroservis, memberikan kontrol yang lebih baik dan meningkatkan keamanan serta observabilitas sistem.


## Instalasi

### Unduh Istio

``` bash
curl -L https://istio.io/downloadIstio | sh -
```


### Pindah Folder

``` bash
cd istio-1.20.0/
```


### Set PATH

``` bash
export PATH=$PWD/bin:$PATH
```


### Instalasi

``` bash
istioctl install --set profile=demo -y
```

``` bash
root@neocom-dev:~/mesh/istio-1.20.0# istioctl install --set profile=demo -y
✔ Istio core installed
✔ Istiod installed
✔ Egress gateways installed
✔ Ingress gateways installed
✔ Installation complete
Made this installation the default for injection and validation.
```


## Konfigurasi

### Namespace

``` bash
kubectl label namespace default istio-injection=enabled
```

### Manifest bookinfo.yaml

``` yaml
cat <<EOF > bookinfo.yaml
# Details service
apiVersion: v1
kind: Service
metadata:
  name: details
  labels:
    app: details
    service: details
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: details
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-details
  labels:
    account: details
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: details-v1
  labels:
    app: details
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: details
      version: v1
  template:
    metadata:
      labels:
        app: details
        version: v1
    spec:
      serviceAccountName: bookinfo-details
      containers:
      - name: details
        image: docker.io/istio/examples-bookinfo-details-v1:1.17.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
        securityContext:
          runAsUser: 1000
---
# Ratings service
apiVersion: v1
kind: Service
metadata:
  name: ratings
  labels:
    app: ratings
    service: ratings
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: ratings
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-ratings
  labels:
    account: ratings
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ratings-v1
  labels:
    app: ratings
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ratings
      version: v1
  template:
    metadata:
      labels:
        app: ratings
        version: v1
    spec:
      serviceAccountName: bookinfo-ratings
      containers:
      - name: ratings
        image: docker.io/istio/examples-bookinfo-ratings-v1:1.17.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
        securityContext:
          runAsUser: 1000
---
# Reviews service
apiVersion: v1
kind: Service
metadata:
  name: reviews
  labels:
    app: reviews
    service: reviews
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: reviews
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-reviews
  labels:
    account: reviews
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v1
  labels:
    app: reviews
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v1
  template:
    metadata:
      labels:
        app: reviews
        version: v1
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v1:1.17.0
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
        securityContext:
          runAsUser: 1000
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v2
  labels:
    app: reviews
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v2
  template:
    metadata:
      labels:
        app: reviews
        version: v2
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v2:1.17.0
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
        securityContext:
          runAsUser: 1000
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v3
  labels:
    app: reviews
    version: v3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v3
  template:
    metadata:
      labels:
        app: reviews
        version: v3
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v3:1.17.0
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
        securityContext:
          runAsUser: 1000
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
---
# Productpage services
apiVersion: v1
kind: Service
metadata:
  name: productpage
  labels:
    app: productpage
    service: productpage
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: productpage
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-productpage
  labels:
    account: productpage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage-v1
  labels:
    app: productpage
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: productpage
      version: v1
  template:
    metadata:
      labels:
        app: productpage
        version: v1
    spec:
      serviceAccountName: bookinfo-productpage
      containers:
      - name: productpage
        image: docker.io/istio/examples-bookinfo-productpage-v1:1.17.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        securityContext:
          runAsUser: 1000
      volumes:
      - name: tmp
        emptyDir: {}
---
EOF
```

### Deploy Manifest
Berkas manifest berisi pendefinisian Kubernetes service, serviceaccount, dan deployment untuk 4 services (details, ratings, reviews dan productpage).

``` bash
kubectl apply -f bookinfo.yaml
```

``` bash
root@neocom-dev:~/mesh/istio-1.20.0# kubectl apply -f bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
```


### Cek Service

``` bash
kubectl get svc
```

``` bash
root@neocom-dev:~/mesh/istio-1.20.0# kubectl get svc
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.107.170.235   <none>        9080/TCP   76s
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP    6h51m
productpage   ClusterIP   10.101.194.199   <none>        9080/TCP   66s
ratings       ClusterIP   10.100.25.25     <none>        9080/TCP   75s
reviews       ClusterIP   10.105.170.2     <none>        9080/TCP   74s
```


### Cek Pod

``` bash
kubectl get po
```

Pastikan semua Pod sudah Running, kalau belum, tunggu dulu.

``` bash
root@neocom-dev:~/mesh/istio-1.20.0# kubectl get po

NAME                              READY   STATUS    RESTARTS   AGE
details-v1-65fbc6fbcf-rd9zl       2/2     Running   0          48m
productpage-v1-7598cf9479-mbhsn   2/2     Running   0          48m
ratings-v1-7fcc55c79f-dwvhp       2/2     Running   0          48m
reviews-v1-66c7d54957-5l74h       2/2     Running   0          48m
reviews-v2-844777b87c-m874h       2/2     Running   0          48m
reviews-v3-b48665d5c-7chzq        2/2     Running   0          48m
```


## Verifikasi 

``` bash
kubectl get pods -l app=ratings
```

``` bash
root@neocom-dev:~/mesh/istio-1.20.0# kubectl get pods -l app=ratings
NAME                          READY   STATUS    RESTARTS      AGE
ratings-v1-7fcc55c79f-dwvhp   1/1     Running   2 (14s ago)   55m
```


``` bash
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

``` bash
<title>Simple Bookstore App</title>
```

Aplikasi berhasil dideploy ke Kubernetes cluster.


# ISTIO INGRESS GATEWAY

**Istio Ingress Gateway** adalah komponen dari Istio yang bertanggung jawab untuk mengelola lalu lintas eksternal yang masuk ke dalam kluster Kubernetes melalui protokol HTTP atau HTTPS. Istio Ingress Gateway berperan sebagai pintu gerbang (gateway) untuk lalu lintas eksternal dan memfasilitasi kontrol, manajemen, serta keamanan lalu lintas masuk ke dalam kluster.

Beberapa fitur dan fungsi utama dari Istio Ingress Gateway meliputi:

1. **Routing:** Istio Ingress Gateway memungkinkan pengkonfigurasian aturan rute yang fleksibel untuk menentukan cara lalu lintas harus diarahkan ke layanan-layanan yang ada dalam kluster.

2. **Load Balancing:** Gateway dapat melakukan pembagian beban lalu lintas ke beberapa instance layanan, meningkatkan ketahanan dan ketersediaan.

3. **TLS Termination:** Istio Ingress Gateway mendukung terminasi TLS, memungkinkan penggunaan HTTPS untuk mengamankan lalu lintas masuk ke dalam kluster.

4. **Authentication:** Gateway dapat digunakan untuk menerapkan kebijakan otentikasi, memastikan bahwa hanya permintaan yang memenuhi syarat tertentu yang diteruskan ke layanan-layanan di dalam kluster.

5. **Authorization:** Istio Ingress Gateway memungkinkan implementasi kebijakan otorisasi yang mengontrol akses ke layanan-layanan berdasarkan aturan tertentu.

6. **Observability:** Gateway menyediakan kemampuan untuk memantau lalu lintas masuk ke dalam kluster dan mengumpulkan metrik serta logging yang terkait.

7. **Rate Limiting:** Pengguna dapat menerapkan pembatasan tingkat (rate limiting) pada lalu lintas yang masuk untuk mencegah serangan atau penggunaan sumber daya yang tidak semestinya.

Istio Ingress Gateway biasanya ditempatkan di luar kluster Kubernetes dan dapat diakses dari internet. Ini membuatnya menjadi titik masuk yang sentral untuk lalu lintas eksternal, dan keberadaannya memungkinkan pengelolaan dan pengendalian lalu lintas yang lebih canggih dibandingkan dengan solusi ingress standar yang disediakan oleh Kubernetes.

Penting untuk dicatat bahwa Istio Ingress Gateway bukan satu-satunya pilihan untuk mengelola lalu lintas masuk dalam Istio. Ada juga Istio Gateway yang dapat digunakan untuk mengonfigurasi lalu lintas masuk dan keluar, termasuk untuk kasus penggunaan yang melibatkan lalu lintas antar layanan di dalam kluster.


## Konfigurasi
Kita telah berhasil deploy aplikasi Bookinfo. Namun, aplikasi tersebut belum bisa diakses dari luar sehingga kita perlu membuat Ingress Gateway yang berfungsi mengelola lalu lintas jaringan yang masuk ke arsitektur service mesh.

### Manifest bookinfo-gateway.yaml

``` yaml
cat <<EOF > bookinfo-gateway.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
EOF
```


### Deploy Manifest

``` bash
kubectl apply -f bookinfo-gateway.yaml
```


### Validasi

``` bash
istioctl analyze
```


### Minikube Tunnel 

Buka Terminal baru

``` bash
minikube tunnel
```


### INGRESS_HOST

Beda terminal dengan minikube tunnel

``` bash 
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
```


### Validasi

``` bash
echo "$INGRESS_HOST" "$INGRESS_PORT" "$SECURE_INGRESS_PORT"
```



### GATEWAY_URL

``` bash 
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```


### Validasi

``` bash
echo "http://$GATEWAY_URL/productpage"
```

``` bash 
http://10.108.210.78:80/productpage
```

Akses di browser. Aplikasi Bookinfo berhasil di-deploy dan dapat diakses dari luar cluster berkat bantuan Istio Ingress Gateway.