# study-ansible

> 배포 자동화 툴

## Good Point
- 사용하기 쉬움(스크립트 방식이 아닌 설정기반)
- 멀티플랫폼지원
- Agent 기반이 아닌 SSH 기반으로 스크립트 배포 없이 관리 서버에서 실행 가능
- 멱등성(여러 번 적용하더라도 결과동일) 보장하는 모듈지원
- 순차 실행뿐 아니라 병렬 실행 지원
- 내부에서 JSON으로 통신 파이썬이 아닌 다른 언어로도 모듈 개발 가능

## Link
### Official
- [ANSIBLE](http://www.ansible.com/)
- [Docs](http://docs.ansible.com/)
- [ansible-examples](https://github.com/ansible/ansible-examples)
- [Galaxy](https://galaxy.ansible.com/)

### Documents
- [raymii.org](https://raymii.org/s/tags/ansible.html)

### Tutorial
- [ansible-tuto](https://github.com/leucos/ansible-tuto)

## Book
- [Ansible 설정 관리](http://www.yes24.com/24/Goods/17833244?Acode=101)
![](http://image.yes24.com/momo/TopCate500/MidCate003/49634478.jpg)

## Word
- 컨트롤러장비
  - ansible이나 playbook을 실행 시키는 장비
- 관리장비
  - ansible로 원격으로 관리하는 장비
- 인벤토리
  - 관리장비 그룹이나 리스트를 갖고있는 것
- 플레이

## Install
### OSX
```
brew install ansible
```

### 도움말
```
ansible-doc -l
ansible-doc file
```

## Playbook
- ansible script 모음 하나 이상의 플레이로 구성

```
ansible-playbook example-play.yml
```

### target
- 플레이가 실행될 장비와 어떻게 플레이가 실행될 것인지 정의
- SSH 사용자 이름과 다른 SSH와 관계된 설정을 지정

```
- hosts: webservers
  users: root
  sudo: yes // root 권한 획득을 위해 사용
  user: flynn // sudo 실행 전에 장비에 연결한 사용자 이름 정의
  sudo_user: root // sudo 사용을 시도하여 되려는 사용자 정의
  connection: ssh // 원격장비 연결하기 위해 사용할 수 있는 전송 방법 정의(ssh, paramiko, local)
  gather_facts: yes // setup 모듈 결과 사용여부 태스크 수행시간을 줄일 수 있음
```

### vars
- 기본값 유지가 되지 않음, 장비 팩트나 인벤토리에서 설정한 변수로 인해 오버라이드 됨
- 플레이에서 실행할 때 쓰일 사용 가능한 변수를 정의

```
vars:
 nginx_version: 1.9.1
 motd_warning: 'WARNING: Use by Flynn Only'
 testservers: yes
```
변수 파일을 통해 로드
```
vars_files:
 /conf/test.yml
```

사용자 프롬프트를 통해서 받음
```
vars_prompt:
 - name:'https_passpharase'
   prompt: 'Key Passphrase'
   private: yes
```

### task
- 앤시블을 이용하여 실행하고 싶은 모든 모듈을 순서대로 열거
- name은 필수 값은 아니다. 하지만 유지보수를 위해서 작성하는 것을 추천
- 콘솔에 name이 있으면 함께 출력됨 없으면 action 내용 출력

```
tasks:
// action: 모듈명령
 - name: install nginx
   action: yum name=nginx state=installed

// ansible이 추천 하는 방식
// module이름: 필요한 인수나열
 - name: configure nginx
   copy: src=files/nginx.conf dest=/etc/nginx/nginx.conf

// 인수 여러줄로 나열
 - name: restart nginx
   service:
    name:nginx
    state: restarted
```

### handler
- 태스크와 동일한 형식 단 태스크나 핸들러에 의해서만 호출됨
- 태스크 리스트가 종료 된 다음 호출됨
- 핸들러가 여러번 호출 되어도 한번만 수행됨
- 태스크의 변경이 발생할때만 호출됨
- 각 핸들러는 하나의 모듈만 수행됨
- 태스크에서 여러 핸들러를 호출 할수 있음

```
handlers:
- name: restart nginx
  action: service name=nginx state=restarted
```

## Modules
- [ping](http://docs.ansible.com/ansible/ping_module.html)
  - ansible 동작이 잘되는지 확인
  - ```ansible machine-name -m ping```
- [setup](http://docs.ansible.com/ansible/setup_module.html)
  - 관리 장비 정보 반환
  - ```ansible machine-name -m setup```
- [file](http://docs.ansible.com/ansible/file_module.html)
  - file관련 모듈 읽기/쓰기/삭제/변경 가능
  - ```ansible machine-name -m file -a 'path=/etc/fstab'```
  - ```ansible machine-name -m file -a 'path=/tmp/test state=directory mode=0700 owner=root'```
  - ```ansible machine-name -m file -a 'path=/tmp/test state=absent'```
- [copy](http://docs.ansible.com/ansible/copy_module.html)
  - 컨트롤러 장비에서 관리 장비로 파일 복사
  - ```ansible machinename -m copy -a 'src=/etc/fstab dest=/tmp/fstab'```
- [command](http://docs.ansible.com/ansible/command_module.html)
  - 관리 장비에서 임의의 커맨드를 실행 할 수 있음
  - 멱등성 보장이 어려움(제한적으로 지원 ex> creates, removes)
  - ```ansible machinename -m command -a 'rm -rf /tmp/testing removes=/tmp/testing'```
- [shell](http://docs.ansible.com/ansible/shell_module.html)
  - 리다이렉션, 파이프 또는 백그라운드 작업을 사용가능
  - 멱등성을 제한적으로 지원(ex > creates 인수만 지원)
  - ```ansible machinename -m shell -a '/opt/testapp/install.sh > /var/log/testapp.log creates=/var/log/testapp.log'```
- [service](http://docs.ansible.com/ansible/service_module.html)
  - 서비스 관련 모듈
- [template](http://docs.ansible.com/ansible/template_module.html)
  - 템플릿을 설정할 수 있는 모듈
  - [jinja2](http://jinja.pocoo.org/) 템플릿을 지원
  - ```template: src=templates/named.conf.j2 dest=/etc/named.conf owner=root group=named mode=0640```
- [set_fact](http://docs.ansible.com/ansible/set_fact_module.html)
  - fact 추가
  - ```set_fact: innodb_buffer_pool_size_mb="{{ ansible_memtotal_mb / 2 }}```
- [pause](http://docs.ansible.com/ansible/pause_module.html)
  - 플레이북 실행동안 일시정지를 할수 있는 모듈
  - 사용자 입력으로 진행여부 확인
    - ```pause: prompt="Wanring! Enter to continue CTRL-C a to quit"```
  - 특정 시간만 대기
    - ```pause: seconds=30```
- [wait_for](http://docs.ansible.com/ansible/wait_for_module.html)
  - 특정 TCP 포트를 폴링, 원격 연결 수락될때까지 대기
  - 폴링은 원격 장비에서 이루어짐
  - ```wait_for: port=8080 state=started```
- [assemble](http://docs.ansible.com/ansible/assemble_module.html)
  - 관리 장비에서 여러 개의 파일을 합치고 관리 장비의 다른 파일로 저장한다.
  - ```assemble: src=/opt/sshkeys dest=/root/.ssh/authorized_keys owner=root group=root mode=0700```
- [add_host](http://docs.ansible.com/ansible/add_host_module.html)
  - 동적으로 플레이에 새로운 장비를 추가
  - ```add_host:```

## group_by
- 동적으로 그룹 생성
```
---
- name: Create operating system group
  hosts: all
  tasks:
  - group_by: key=os_{{ ansible_distribution }}
  - name: Run on CentOS hosts only
    hosts: os_CentOS
    tasks:
    - name: Install Nginx
      yum: name=nginx state=latest
  - name: Run on Ubuntu hosts only
    hosts: os_Ubunutu
    tasks:
    - name: Install Nginx
      apt: pkg=nginx state=latest
```

## 병렬로 작업 실행
- task에 async와 poll 을 추가한다.

```
tasks:
 - name: Install mlocate
   yum: name=mlocate state=installed
 - name: Run updatedb
   command: /usr/bin/updatedb
   async: 300 // 커맨드가 완료될 때까지 앤시블이 기다려 줄 수 있는 최대 값
   poll: 10 // 커맨드가 완료될 때를 점검하기 위해 얼마나 자주 폴링
```
poll 0 // 앤시블이 완료를 기다리지 않음
async 0 // 작업을 완료할 때까지 기다림

## Looping
- 반복적인 태스크를 처리 할 때 사용 with\_items, with\_fileglob을 활용
- 비슷한 설정으로 모듈을 많이 반복
- 하나의 목록인 팩트의 모든 값을 반복
- 하나의 큰 파일로 결합할 assemble 모듈과 함께 나중에 사용할 수 있는 많은 파일을 생성하기 위해 사용
- glob 패턴 매칭을 사용 중인 디렉토리를 복사하기 위해 with\_fileglob를 사용

```
tasks:
 - name: Secure config files
   file: path=/etc/{{ item }} mode=0600 owner=root group=root
   with_items:
    - my.cnf
    - shadow
    - fstab
```
// lookup 플러그인
```
tasks:
 - name: Upload public keys
   copy: src={{ item }} dest=/root/.sshkeys mode=0600 owner=root group=root
   with_fileglob:
    - keys/*.pub
```

## 조건절
- 특정 조건에서만 동작하도록 한다.
- 운영체제의 차이를 극복하며 작업
- 사용자에게 프롬프트를 보여 요청한 액션만 수행
- 뭔가를 변경하지 않고 실행하기에 오랜 시간이 소요될 수 있는 모듈을 피해 성능을 개선
- 특정한 파일이 존재하는 시스템의 변경을 거절
- 사용자가 작성한 스크립트가 이미 실행 중인지 체크

```
tasks:
 - name: Install VIM via yum
   yum: name=vim-enhanced state=installed
   when: ansible_os_family == "RedHat"
   
 - name: Install VIM via apt
   apt: name=vim state=installed
   when: ansible_os_family == "Debian"
   
 - name: Unexpected OS family
   debug: msg="OS Family {{ ansible_os_family }} is not supported" fail=yes
   when: not ansible_os_family == "RedHat" or ansible_os_family == "Debian"
```
## 태스크 위임
- 다른 장비에서 액션 수행이 필요한 경우 사용
- 배포 전에 로드 밸런서로부터 장비를 제거하기
- 변경하려는 서버에서 트래픽을 빼는 DNS를 변경하기
- 저장 장치의 iSCI 볼륨을 생성하기
- 네트워크 작업 바깥에 접근을 점검할 수 있게 외부 서버를 사용하기

```
tasks:
 - name: Get config
   get_url: dest=configs/{{ ansible_hostname }} force=yes url=http://{{ ansible_hostname }}/diagnostic/config
   delegate_to: localhost
```
// localhost에 위임하는 경우는 ```local_action```이 있음
```
tasks:
 - name: Get config
   local_action: get_url dest=configs/{{ ansible_hostname }} force=yes url=http://{{ ansible_hostname }}/diagnostic/config
```

## 추가변수
### hostvars
- 현재 플레이가 다뤘던 모든 장비에 대한 변수를 얻을수 있음

```
{{ hostvars.[hostname].ansible_default_ipv4.address }}
```

### groups
- 인벤토리 그룹에 의해 구분짓는 인벤토리의 모든 장비의 목록을 포함, 모든 장비와 모든 그룹에 반복할 수 있음

```
- name: Create a user for all app servers
  with_items: groups.appservers
  mysql_user: name=flynn password=test host={{ hostvars.[item].ansible_eth0.ipv4.address }} state present
```

### group_names
- 현재 장비가 포함된 모든 그룹의 이름으로 문자열 목록을 포함

```
tasks:
 - name: For secure machines
   set_fact: sshconfig=files/ssh/sshd_config_secure
   when: "'secure' in group_names"
 - name: For non-secure machines
   set_fact: sshconfig=files/ssh/sshd_config_default
   when: "'secure' not in group_names"
 - name: Copy over the config  
   copy: src={{ sshconfig }} dest=/tmp/sshd_config
```

### inventory_hostname
- 인벤토리에서 저장된 서버의 장비 이름을 저장


### inventory\_hostname\_short
- 인벤토리에서 저장된 서버의 장비 이름을 저장 중 첫 번째 점(.)까지의 캐릭터만 포함

### inventory_dir
- 인벤토리 파일을 포함하는 디렉토리의 경로 이름

### inventory_file
- inventory_dir에서 인벤토리 파일명이 추가된 이름

## 변수로 파일 찾기
```
copy: src=files/nrpe.{{ ansible_architecture }}.conf dest=/etc/nagios/nrpe.cfg
```
```
name: Get the best match for the machine
copy: dest=/etc/nginx.conf src={{ item }}
first_available_file:
 - files/nginx/{{ ansible_os_family }}-{{ ansible_architecture }}.conf
 - files/nginx/default-{{ ansible_architecture }}.conf
 - files/nginx/default.conf
```
## Environment
- environment로 환경 변수를 추가 할수 있음
- 애플리케이션 인스톨러 실행하기
- shell 모듈을 이용할 때 경로에 추가적인 요소를 추가하기
- 시스템 라이브러리 검색 경로에 포함되지 않은 장소로부터 라이브러리를 로딩하기
- 모듈을 실행하는 동안 LD_PRELOAD 해킹을 이용하기

```
name: Upload the file
shell: aws s3 put-object --bucket=my-test-bucket --key={{ ansible_hostname }}/fstab --body=/etc/fstab --region=eu-west-1
environment:
 AWS_ACCESS_KEY_ID: XXXXXXXX
 AWS_SECRET_ACCESS_KEY: XXXXXXXX
```

## 외부 데이터 검색
- conf.d 방식의 디렉토리에 모든 아파치 설정 디렉토리를 복사하기
- 플레이북이 무엇을 하는지 조정하기 위한 환경 변수 얻기
- DNS TXT 레코드에서 설정 얻기
- 커맨드 결과를 얻어 변수로 저장

직접 호출 방식
```
name: Download file
get_url: dest=/var/tmp/file.tar.gz url=http://server/file.tar.gz
environment:
 http_proxy: "{{ lookup('env', 'http_proxy') }}"
```

with_* 사용하는 방식
```
name: Register the webapp farm
local_action: add_host name={{ item }} groupname=webapp
with_sequence: start=1 end=10 format=webapp%02x
```

## 결과 저장
- register 를 사용해서 결과를 저장한다.
- fetch 모듈로 원격 디렉토리에서 파일 목록을 얻고 모두 다운로드하기
- 핸들러가 실행하기 전, 앞 태스크가 변경할 때 태스크를 실행하기
- 원격 장비의 SSH키 내용을 얻어 known_hosts 파일을 만들기

```
- name: Get /tmp info
  file: dest=/tmp state=directory
  register: tmp
  
- name: Set mode on /var/tmp
  file: dest=/tmp/subtmp mode={{ tmp.mode }} state=directory
```

## Debug
### debug
- 변수의 현재값 보기 좋음, msg는 출력 내용, fail은 진행여부 yes인 경우 멈춤

```
debug: msg="{{ item }}"
with_items: ansible_interfaces
```

### verbose
- verbose모드일 경우 앤시블이 실행 이후 각 모듈에서 리턴된 모든 값을 출력
- register 키워드를 사용하고 있다면 유용

```
ansible-playbook --verbose playbook.yml
```

### check / diff
- 실제 관리장비에 어떠한 변화를 주지 않고 실행 참고로 diff는 template 모듈이 생성한 변화만 출력

### pause
- 잠시 멈출때 사용


## Include
- 중복을 없애기 위해 사용

### var include

### task include
```
# usersetup.yml
# Requires a user variable to specify user to setup
- name: Create user account
  user: name={{ user }} state=present

- name: Make user SSH config dir
  file: path=/home/{{ user }}/.ssh owner={{ user }} group={{ user }} mode=0600 state=directory

- name: Copy in public key
  copy: src=keys/{{ user }}.pub dest=/home/{{ user }}/.ssh/authorized_keys mode=0600 owner={{ user }} group={{ user }}
```
```
tasks:
  - include: usersetup.yml user={{ item }}
    with_items:
      - niceilm
      - flynn
```
### handler include
```
handlers:
- include: sendmailhandlers.yml
```

### playbook include
```
---
- include "drfailover.yml"
- include "upgradeapp.yml"
- include "drfailback.yml"

- name: Notify management
  host: local
  tasks:
  - local_action: mail to="niceilm@naver.com" msg='The application has been upgraded and is now live'

- include "drupgrade.yml"
```

### Role

### Tag

### ansible-pull

## Error Handling In Playbooks
- http://docs.ansible.com/ansible/playbooks_error_handling.html
```
    - name: cache
      apt: purge=yes name=lxc-docker
      ignore_errors: yes
```
