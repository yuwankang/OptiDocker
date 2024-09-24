# 🚀OptiDocker 도커이미지 최적화 프로젝트 🚀

# 🎯 프로젝트 개요
> Docker 이미지를 최적화하는 것은 애플리케이션의 성능과 보안을 향상시키고, 배포 시간을 단축시키며, 리소스 사용을 최적화하기 위해 매우 중요합니다. 이러한 최적화 과정은 다음과 같은 이점을 제공합니다:)
### 비용 절감 💰: 최적화된 이미지는 리소스 사용량을 줄여 클라우드 비용 절감에 기여합니다.
### 빠른 배포 ⏱️: 이미지 크기가 작아지면 배포 속도가 빨라지고, CI/CD 파이프라인이 더 효율적으로 작동합니다.
### 보안 향상 🔒: 불필요한 파일과 의존성을 제거하여 보안 공격의 표면을 줄입니다.


###  📁 폴더 구조
  
![](https://velog.velcdn.com/images/yuwankang/post/311b2697-48db-4d69-b0d4-afcd87f8e739/image.png)
## 🐳 비교를 위한 기본 빌드
```
FROM openjdk:17
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
RUN javac Main.java
CMD ["java", "Main"]
```
![](https://velog.velcdn.com/images/yuwankang/post/044a0aa0-2e5f-42dd-abcd-ef6b17a34520/image.png)

![](https://velog.velcdn.com/images/yuwankang/post/6b57037b-5e07-4a9c-b945-4b82d92f91e6/image.png)

## 0. Node.js 파일 최적화 하기 🛠️
> 프로젝트 초반 Node.js 파일 최적화를 하려 하였지만 한국은 **JAVA** 강국 이기 때문에 JAR 파일을 만들어 진행 하였습니다.
![image](https://github.com/user-attachments/assets/4b21b586-9985-47a2-b969-0652464e87d9)
![image](https://github.com/user-attachments/assets/98d3eece-9c85-4840-b014-7468a65d778a)

## 1. Minimal Base Images 🌱
- Alpine 기반 이미지를 사용하는 방식입니다.
```
# Minimal Base Image
FROM openjdk:17-alpine

# 작업 디렉토리 설정
WORKDIR /usr/src/myapp

# 모든 파일 복사
COPY . .

# 애플리케이션 컴파일
RUN javac Main.java

# 애플리케이션 실행
CMD ["java", "Main"]

```
- 테스트 코드
```
docker build -t my-docker-project_minimal .
```
![](https://velog.velcdn.com/images/yuwankang/post/b80715dd-9330-4e7d-8c8c-611267903f26/image.png)
### 강점
- 🔹 작은 이미지 크기: Alpine 이미지를 사용하여 전체 이미지 크기를 줄입니다.
- 🔹 보안: Alpine은 최소한의 패키지만 포함되어 있어 공격 표면이 줄어듭니다.


## 2. Minimize Layers 📉
- 다양한 RUN 명령을 합쳐서 레이어를 줄이는 방식입니다.
```
# Minimize Layers (Alpine 사용)
# 여러 RUN 명령을 결합하여 레이어를 최소화합니다.
FROM openjdk:17-alpine

# 작업 디렉토리 설정
WORKDIR /usr/src/myapp

# 모든 파일 복사
COPY . .

# 애플리케이션 컴파일과 메시지를 결합하여 단일 RUN 명령으로 레이어 최소화
RUN javac Main.java && echo "Compilation complete"

# 애플리케이션 실행
# Java 애플리케이션을 실행합니다.
CMD ["java", "Main"]
```
- 테스트 코드
```
docker build -t my-docker-project_minimize .
```
![](https://velog.velcdn.com/images/yuwankang/post/14637d14-8e14-444a-abac-6a507efa5a42/image.png)

![](https://velog.velcdn.com/images/yuwankang/post/0b0f2524-d994-4c68-b488-d1e8c1611a98/image.png)
### 강점
- 🔹 레이어 최적화: 여러 명령을 하나의 RUN 명령으로 결합하여 레이어 수를 줄이고 이미지 크기를 최적화합니다.
- 🔹 빠른 빌드 시간: 불필요한 레이어를 줄여 빌드 시간을 단축합니다.

## 3. Multi-Stage Builds🔄
- 빌드 환경을 분리하여 최종 이미지의 크기를 줄이는 방식입니다.

```
# 1단계: 빌드 환경 설정
FROM openjdk:17-alpine AS build

# 작업 디렉토리 설정
WORKDIR /usr/src/myapp

# 모든 파일 복사
COPY . .

# 애플리케이션 컴파일
RUN javac Main.java

# 2단계: 최종 이미지
FROM gcr.io/distroless/java17

# 작업 디렉토리 설정
WORKDIR /app

# 빌드 단계에서 생성한 클래스 파일 복사
COPY --from=build /usr/src/myapp/Main.class .

# 애플리케이션 실행
CMD ["java", "Main"]

```
- 테스트 코드
```
docker build -t my-docker-project_multistage .
```
### 강점
- 🔹 최종 이미지 크기 감소: 빌드 단계와 실행 단계를 분리하여 최종 이미지에서 불필요한 파일과 의존성을 제거합니다.
- 🔹 최적화된 보안: Distroless 이미지를 사용하여 공격 표면을 줄입니다.

![](https://velog.velcdn.com/images/yuwankang/post/658008c0-5b2f-44fe-b51f-a85c5afe7de5/image.png)
![](https://velog.velcdn.com/images/yuwankang/post/b1b613d1-7d04-40d6-9265-6046641f167b/image.png)


## 4. Use .dockerignore 📜
> .dockerignore 파일을 사용하여 Docker 이미지에 불필요한 파일과 디렉토리를 제외합니다.
> 폴더 구조 생성: 프로젝트의 루트 디렉토리에 .dockerignore 파일을 생성합니다.

- .dockerignore 파일 작성:
```
node_modules
Dockerfile*
.git
.github
.gitignore
dist/**
README.md
```
- 테스트 코드
```
docker build -t my-docker-project_with_dockerignore .
```
### 강점
- 🔹 이미지 크기 감소: 불필요한 파일과 디렉토리를 제외함으로써 Docker 이미지의 크기를 줄여서 더 빠른 배포를 가능하게 합니다.
- 🔹 빌드 속도 향상: 필요하지 않은 파일이 제외되면 Docker가 이미지를 빌드하는 시간이 단축되어 효율성이 증가합니다.
- 🔹 보안 강화: 중요한 파일(예: .git 디렉토리, README.md 등)을 이미지에서 제외함으로써 보안 위험을 줄입니다. 이로 인해 민감한 정보가 노출될 가능성이 줄어듭니다.
- 🔹 리소스 절약: Docker 빌드 프로세스 중에 불필요한 파일을 처리하는 데 소모되는 리소스를 줄여 전체적인 시스템 성능을 향상시킵니다.
![image](https://github.com/user-attachments/assets/253091bc-35b6-4ed2-918a-a3af4f11fd57)

## ✨ 결론
> 이 프로젝트는 Docker 이미지를 최적화하여 애플리케이션의 성능, 보안, 그리고 배포 속도를 크게 향상시킵니다. 각 최적화 기법은 서로 다른 장점을 제공하며, 이를 통해 클라우드 비용 절감과 효율적인 리소스 관리를 달성할 수 있습니다. 최종적으로, 이러한 최적화 작업은 DevOps 환경에서 지속적인 통합 및 배포(CI/CD) 프로세스를 개선하여 현대 소프트웨어 개발에 중요한 기여를 합니다.

