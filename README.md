# Disconnect Openshift Container Platform 구축 및 CI/CD Pipeline

## 프로젝트 목표

**1. PaaS 클라우드 구축**

일반적으로 고객사들은 자사의 보안을 위해 외부와 인터넷이 단절된, 즉 disconnected 환경에서 클라우드 Platform을 구축한다. 본 프로젝트를 통해 이러한
 Disconnected 환경에서 Openshift 기반의 PaaS 클라우드 아키텍쳐를 수립하고 환경을 구축한다. PaaS 클라우드를 구축하기 위한 사전 준비 사항 및 고려 사항을 파악하고 일반적인 아키텍처에 대해 이해한다. 

**2. DevSecOps Pipeline 구성**

환경 및 데이터 보안, CI/CD 프로세스 및 보안을 고려한 DevSecOps Pipeline을 구성하고 Jenkins, Gitlab, API, Quay 등을 연계하여 PaaS 클라우드 환경에서 빌드 및 배포의 자동화를 구성한다.  
<br></br>

## 주요 내용

**1. Disconnected 환경에서의 기초 인프라 Architecture 설계 및 구축**
- 기반 서비스 (DNS, Haproxy, Chrony, Yum Repository, Image-Registry) 설계와 구축
- 애플리케이션 서비스 (Gitlab, Image Registry) 설계와 구축

**2. 표준 컨테이너 및 오케스트레이션 기술 이해**
- 프라이빗 PaaS 클라우드 (Openshift 4) 아키텍쳐 설계 및 구축
- Master, Infra, Router, App Node 설계와 구축

**3. Git, Jenkins, Quay 등을 활용한 DevSecOps Pipeline 설계와 구축**
- 사용자 ID 권한 부여 및 Access 제어
- 컨테이너 분리 및 네트워크 분리
- 컨테이너 이미지 스캔

<br></br>
## 프로젝트 인프라 구성 환경

- 노트북 4대, Hub 1대,  LAN Cable 4개
    - 노트북 스펙
        - Ubuntu 18.04
        - 5 Core
        - RAM 16 GB
        - SSD 512 GB
        - Hypervisior : KVM
- 네트워크 통신 방법
    - Host < — > 다른 Host의 VM : Bridge 통신
    - Host < — > 본인 Guest VM : Host-Only 통신

<img src="/image/ProjectEnvironment.png" width="500" height="500">

# Architecture

## 1. 각 노드별 Resource, IP Settings과 역할

- Cluster Name : redhat2
- Base Domain : cccr.local
- Openshift Service network : 172.30.0.0/16
- cluster network ( Pod ) : 10.1.0.0/16



<table>
<thead>
    <tr>
        <th>구분</th>
        <th>서버명</th>
        <th>OS 구분</th>
        <th>Hostname<br>(Domain).redhat2.cccr.local</th>
        <th>IP</th>
        <th>vcpu<br>(core)</th>
        <th>Memory<br>(GB)</th>
        <th>OS</th>
        <th>contaimer<br>Runtime</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td rowspan=7>컨테이너 <br>관리서버</td>
        <td>BootStrap</td>
        <td>RHCOS 4.4</td>
        <td align="center">bootstrap</td>
        <td>10.10.10.10</td>
        <td align="center">4</td>
        <td align="center">16</td>
        <td>120</td>
        <td>100</td>
    </tr>
    <tr>
        <td>Master #1</td>
        <td>RHCOS 4.4</td>
        <td align="center">master-1</td>
        <td>10.10.10.11</td>
        <td align="center">4</td>
        <td align="center">8</td>
        <td>100</td>
        <td>100</td>
    </tr>
    <tr>
        <td>Master #2</td>
        <td>RHCOS 4.4</td>
        <td align="center">master-2</td>
        <td>10.10.10.12</td>
        <td align="center">4</td>
        <td align="center">8</td>
        <td>100</td>
        <td>100</td>
    </tr>
    <tr>
        <td>Master #3</td>
        <td>RHCOS 4.4</td>
        <td align="center">master-3</td>
        <td>10.10.10.13</td>
        <td align="center">4</td>
        <td align="center">8</td>
        <td>100</td>
        <td>100</td>
    </tr>
    <tr>
        <td>Infra #1 </td>
        <td>RHCOS 4.4</td>
        <td align="center">infra-1</td>
        <td>10.10.10.14</td>
        <td align="center">4</td>
        <td align="center">8</td>
        <td>100</td>
        <td>100</td>
    </tr>
    <tr>
        <td>Infra #2</td>
        <td>RHCOS 4.4</td>
        <td align="center">infra-2</td>
        <td>10.10.10.15</td>
        <td align="center">4</td>
        <td align="center">8</td>
        <td>100</td>
        <td>100</td>
    </tr>
    <tr>
        <td>Router</td>
        <td>RHCOS 4.4</td>
        <td align="center">router</td>
        <td>10.10.10.16</td>
        <td align="center">2</td>
        <td align="center">3</td>
        <td>100</td>
        <td>100</td>
    </tr>
    <tr>
        <td>주변시스템</td>
        <td>Bastion</td>
        <td>RHCOS 4.4</td>
        <td align="center">bastion</td>
        <td>10.10.10.17</td>
        <td align="center">4</td>
        <td align="center">8</td>
        <td>100</td>
        <td>100</td>
    </tr>
    <tr>
        <td rowspan=2>컨테이너 <br> 업무서버 </td>
        <td>Service #1</td>
        <td>RHCOS 4.4</td>
        <td align="center">service-1</td>
        <td>10.10.10.18</td>
        <td align="center">2</td>
        <td align="center">4</td>
        <td>100</td>
        <td>100</td>
    </tr>
    <tr>
        <td>Service #2</td>
        <td>RHCOS 4.4</td>
        <td align="center">service-2</td>
        <td>10.10.10.19</td>
        <td align="center">2</td>
        <td align="center">4</td>
        <td>100</td>
        <td>100</td>
    </tr>

</tbody>
</table>

- Bastion Node : Openshift 4 Container Platform 구축의 기반 역할
    - DNS, HAProxy, Image Registry, Chrony, Yum Repository 서버 구축
- Master Node : Openshift 4 Control Plane Node, 고가용성을 위한 3중화
- Router Node : Application이 배포될 Service Node로 Routing Node
- Infra Node : Logging, Monitoring 를 위한 Node
- Service Node : 실질적인 Application이 배포되는 Node

## 2. 논리적 Architecture

![logical](./images/logical.png)

## 3. 물리적 Architecture

- Laptop 4대에 VM의 리소스 할당량을 고려하여 배치하였습니다.  한다. / 합니다.
- Com #3의 경우 Bootstrap은 Master Node 설치 후 삭제해도 되므로 삭제한 후 Infra #2, Router Node를 올렸습니다.

![physical](./images/physical.png)
