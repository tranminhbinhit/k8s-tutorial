# Hành với Docker, K8s
## Tạo ứng dụng api .net 6.0 trên visua studio code
    1. Tạo Project netcore-btest
        dotnet new webapi -n netcore-btest
        dotnet new webapi -n netcore-btestv -f net6.0
    2. Buil project
        dotnet build
    3. Tạo cors https
        dotnet dev-certs https
    3. Run
        dotnet run

## Publish ứng dụng lên hub docker
### Tạo image từ ứng dụng lên docker (Phải cài Docker Desktop trên máy local)
    1. Tạo file Dockerfile
        ```
            # Get base SDK Image from Microsoft
            FROM mcr.microsoft.com/dotnet/sdk:6.0 as build-env
            WORKDIR /app

            # Copy the CSPROJ file and restore any dependencies (via NUGET)
            COPY *.csproj ./
            RUN dotnet restore

            # Copy the project files and build out release
            COPY . ./
            RUN dotnet publish -c Release -o out

            # Generate runtime image
            FROM mcr.microsoft.com/dotnet/aspnet:6.0
            WORKDIR /app
            EXPOSE 80
            COPY --from=build-env /app/out .
            ENTRYPOINT ["dotnet", "netcore-btestv.dll"]
        ```
    2. Tạo file .dockerignore
        ```
            bin\
            obj\
        ```
    3. Tạo image trên docker desktop
        docker build -t netcore-btestv .

    4. Kiểm tra image đã tạo trên docker
        docker image ls
            => netcore-btestv    latest => Thành công rồi đó

    5. Chạy thử nghiệm ứng dụng trên image docker
        docker run -d -p 8080:80 --name testapp netcore-btestv
            => Create container (Port public:8080, Port private container: 80, ContainerName: testapp, Image: netcore-btestv)
        docker container ls
            => netcore-btestv  0.0.0.0:8080->80/tcp     testapp  => Thành công rồi (Run thử trên http://localhost:8080/WeatherForecast)

### Tiến hành upload image lên docker hub => Free
    1. Tạo tài khoản trên docker, Login tài khoản trên Docker desktop
        https://hub.docker.com/

    2. Map image trên docker desktop local lên hub.docker
        docker tag netcore-btestv tranminhbinhit/netcore-btestv
            (netcore-btestv: Image trên local, tranminhbinhit/netcore-btestv: Image trên Hub)

    3. Push image từ local lên hub.docker đã map ở step 2
        docker push tranminhbinhit/netcore-btestv

# Lấy image từ hub.docker để chạy trên K8s
## Tạo file yaml cho kube đọc
    1. Tải ứng dụng mRemoteNG
        Cấu hình: 
            IP: 147
            Protocal: SSH version 2
            User Name: root
            Password: ******
    2. Tạo file yaml để k8s chạy
        Tạo file "hello-nercore-btestv.yaml" trỏ vào image (image đã tạo trước đó hoặc 1 image nào đó trên hub)
        /etc/kubernetes/binh-manifests/hello-nercore-btestv.yaml
        Lưu ý: port 82 trùng với port của file Dockerfile
        ```
            apiVersion: v1 # Descriptor conforms to version v1 of Kubernetes API
            kind: Pod # Select Pod resource
            metadata:
                name: hello-nercore-btestv # The name of the pod
            spec:
            containers:
                - image: tranminhbinhit/netcore-btestv # Image to create the container
                name: hello-nercore-btestv # The name of the container
                ports:
                    - containerPort: 82 # The port the app is listening on
                    protocol: TCP


        ```
        VD: 
            cd /etc/kubernetes/
            mkdir binh-manifests
            cd binh (TAB) => cd binh-manifests
            vi hello-nercore-btestv.yaml
            (Paste nội dung) 
            :x!
    2.1. Một số lệnh terminal 
        cd <root>               : Vào root
        mkdir <folder name>     : Tạo thư mục
        vi <filename>           : Mở file hoặc tạo
        ESC :q!                 : Thoát không lưu
        ESC :x!                 : Thoát & lưu
        cp <root file> <end root>: Sao chép file
        rm -f <file name>       : Xoá file     
        
## Run yaml 
    1. Chay file yaml
        kubectl apply -f hello-netcore-btestv.yaml
        -- Chay file tren github - chưa dc 
        kubectl apply -f https://raw.githubusercontent.com/tranminhbinhit/docker-compose/master/netcore-cli/netcore-btestv/hello-nercore-btestv.yaml?token=GHSAT0AAAAAAB6MV35Y7FSEQZU7XQZGZW2IZC3D6KQ
    
    2. Forward all server run pods (Pub ra port 3001)
        kubectl port-forward pod/hello-kube-binh 3001:80

    3. Test kết quả (Mở ra 1 terminal khác) 
        curl localhost:3001/WeatherForecast

    4. Note lại 1 số câu lệnh
        ```
            vi hello-kube.yaml
            Run pod
                kubectl apply -f hello-kube.yaml
            Forward all server run pods (Pub ra port 3001)
                kubectl port-forward pod/hello-kube-binh 3001:80
            Xoá pod
                kubectl delete pod hello-kube-binh
            Xem chi tiet app trong pods
                kubectl get pods -owide
            Copy file 
                cp custom-manifests/hello-kube.yaml binh-manifests/hello-nercore-btestv2.yaml
            Xoá file
                rm -f hello-kube.yaml
        ```
        
            


    


        

    







