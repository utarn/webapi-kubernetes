# ตัวอย่างโปรเจค WebAPI บน Kubernetes

ให้สร้างโปรเจคด้วยคำสั่ง

```bash
mkdir mywebapi
cd mywebapi
dotnet new webapi
```

จากนั้น สร้าง Dockerfile

```
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore

COPY . .
RUN dotnet build -c Release -o /app/build

FROM build AS publish
RUN dotnet publish -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "mywebapi.dll"]
```


1. จากนั้นสร้าง Image

```
docker build -t utarn/mywebapi:1.0 .
docker push utarn/mywebapi:1.0
```

ทดสอบการทำงานในเครื่อง

```
 docker run -it -p 8080:80 --name mywebapi --rm utarn/mywebapi:1.0
```

จากนั้นเปิดเบราเซอร์ดูที่ http://localhost:8080/WeatherForecast

เมื่อได้แล้ว สามารถ deployment ใน Kubernetes

```
kubectl apply -f deployment.yaml
```

deploy แล้วแต่ยังเข้าถึงไม่ได้ ต้องทำ service เพื่อเปิด port ก่อน

```
kubectl apply -f service.yaml
```

จากนั้นทดลอง scale ตัวเว็บไซต์ให้เป็น 4 replicas

```
kubectl apply -f deployment2.yaml
```

เข้าไปดู Process ใน WSL
```
wsl -d docker-desktop
ps aux | grep dotnet
```