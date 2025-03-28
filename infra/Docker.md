# Docker

Docker는 VM을 압도적인 성능으로 대체하는 가상화 기술이다. 

빠르고 효율적인 이유는 기본적으로 OS, Hardware 자원을 소프트웨어화 하지 않기 때문이다. 

따라서 일반적인 프로세스처럼 사용하돼, 이 컨테이너들은 도커 엔진이 관리한다고 보면 될 것 같다.

그럼 도커는 어떻게 로컬 머신의 리눅스 OS를 이용할까?

## Cgroup

리눅스에서 제공하는 cgroup을 활용하면, 프로세스간의 격리된 ( 마치 다른 머신 처럼 ) 공간으로 자원을 소모할 수 있다.

2. CPU 제한 (CPU cgroup)
2.1 cgroup 디렉터리 생성
```
mkdir /sys/fs/cgroup/cpu/mycpu
```

2.2 CPU shares 설정
cpu.shares는 프로세스 간 CPU 사용 우선순위를 의미합니다.

기본값은 1024이며, 숫자가 높을수록 우선순위가 높아집니다.

```
echo 512 > /sys/fs/cgroup/cpu/mycpu/cpu.shares
```

2.3 Memory 제한

```
# 최대 메모리를 200MB로 제한
echo 200M > /sys/fs/cgroup/cpu/mycpu/memory.limit_in_bytes
```

내부 태스크들의 메모리도 제한 가능하다.

<br>
도커는 바로 이러한 cgroup의 특성을 활용해 제작됐다.


## +) 도커 이미지 생성

### before
FROM ubuntu:16.04
MAINTAINER subicura@subicura.com
RUN apt-get -y update
RUN apt-get -y install ruby
RUN gem install bundler

### after
FROM ruby:2.3
MAINTAINER subicura@subicura.com

Base Image를 항상 java~~, python ~~ 했으나, 이것들이 전부 before를 간략화하여 제공됐던 것이었다.

- 도커 이미지는 단계별로 캐싱된다.

### before 
COPY . /usr/src/app    # <- 소스파일이 변경되면 캐시가 깨짐
WORKDIR /usr/src/app
RUN bundle install     # 패키지를 추가하지 않았는데 또 인스톨하게 됨 ㅠㅠ

### after

COPY Gemfile* /usr/src/app/ # Gemfile을 먼저 복사함
WORKDIR /usr/src/app
RUN bundle install          # 패키지 인스톨
COPY . /usr/src/app         # <- 소스가 바꼈을 때 캐시가 깨지는 시점 ^0^

캐시가 깨질 부분을 최대한 뒤로 밀면, 속도가 빨라진다.


## 참고 사이트

https://subicura.com/2017/02/10/docker-guide-for-beginners-create-image-and-deploy.html

https://youtu.be/zh0OMXg2Kog?si=Y0mGSeRbY9IFBwXW
