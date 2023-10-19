# Домашнее задание к занятию 1 «Введение в Ansible»

## Подготовка к выполнению

1. Установите Ansible версии 2.10 или выше.
Ответ:
```
[vitalii@fedora 8.1]$ ansible --version
ansible [core 2.14.5]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/vitalii/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/vitalii/.pyenv/versions/3.11.1/lib/python3.11/site-packages/ansible
  ansible collection location = /home/vitalii/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/vitalii/.pyenv/versions/3.11.1/bin/ansible
  python version = 3.11.1 (main, Jan 22 2023, 19:21:33) [GCC 12.2.1 20221121 (Red Hat 12.2.1-4)] (/home/vitalii/.pyenv/versions/3.11.1/bin/python3.11)
  jinja version = 3.1.2
  libyaml = True


```
2. Создайте свой публичный репозиторий на GitHub с произвольным именем.
Ответ:
Создал

3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
Ответ:
Скачал
## Основная часть
1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте значение, которое имеет факт `some_fact` для указанного хоста при выполнении playbook.
Ответ:
```[vitalii@fedora playbook]$ ansible-playbook site.yml -i inventory/test.yml 

PLAY [Print os facts] *********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] ***************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Fedora"
}

TASK [Print fact] *************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP ********************************************************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


Значение 12
```
2. Найдите файл с переменными (group_vars), в котором задаётся найденное в первом пункте значение, и поменяйте его на `all default fact`.
Ответ:
```
[vitalii@fedora all]$ pwd
/home/vitalii/GIT2/NETOLOGY2/8.1/playbook/group_vars/all
[vitalii@fedora all]$ cat examp.yml 
---
  some_fact: all default fact[vitalii@fedora all]$ 

```
3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.
Ответ:
```
docker-compose.yml
version: '3.5'
services:
  centos7:
    image:  centos:7
    restart: unless-stopped
    entrypoint: "sleep infinity"
    networks:
      -  home
  ubuntu:
    image:  ubuntu
    restart: unless-stopped
    entrypoint: "sleep infinity"
    networks:
      -  home
networks:
  home:
    name:  home
    driver:  bridge

vitalii@fedora ~/G/N/8.1 (main)> docker-compose up -d
[+] Running 2/2
 ⠿ Container 81-ubuntu-1  Started                                                                                                                                                                                                     1.3s
 ⠿ Container 81-centos-1  Started                                                                                                                                                                                                     1.3s
vitalii@fedora ~/G/N/8.1 (main)> docker-compose ps
NAME                COMMAND             SERVICE             STATUS              PORTS
81-centos-1         "sleep infinity"    centos              running             
81-ubuntu-1         "sleep infinity"    ubuntu              running       
```
4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйе полученные значения `some_fact` для каждого из `managed host`.
Ответ:
```
root@FEDORA /h/v/N/8/playbook# ansible-playbook site.yml -i inventory/prod.yml 

PLAY [Print os facts] *********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ***************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] *************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP ********************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились значения: для `deb` — `deb default fact`, для `el` — `el default fact`.
Ответ:
```
vvakhromeev@FEDORA ~/N/8/p/group_vars [2]> cat deb/examp.yml 
---
  some_fact: "deb default fact"⏎             

  vvakhromeev@FEDORA ~/N/8/p/group_vars> cat el/examp.yml
---
  some_fact: "el default fact"⏎            
```
6.  Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.
Ответ:
```
root@FEDORA /h/v/N/8/playbook# ansible-playbook site.yml -i inventory/prod.yml

PLAY [Print os facts] *********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ***************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] *************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ********************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```
7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.
Ответ:
```
root@FEDORA /h/v/N/8/p/group_vars [1]# ansible-vault encrypt deb/examp.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful
root@FEDORA /h/v/N/8/p/group_vars# ansible-vault encrypt el/examp.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful

```
8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.
Ответ:
```
root@FEDORA /h/v/N/8/playbook [1]# ansible-playbook site.yml -i inventory/prod.yml

PLAY [Print os facts] *********************************************************************************************************************************************************************************************************************
ERROR! Attempting to decrypt but no vault secrets found

```
9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.
Ответ:
```
root@FEDORA /h/v/N/8/playbook [4]# ansible-doc -t connection -l
ansible.builtin.local          execute on controller                                                                                                                                                                                  
ansible.builtin.paramiko_ssh   Run tasks via python ssh (paramiko)                                                                                                                                                                    
ansible.builtin.psrp           Run tasks over Microsoft PowerShell Remoting Protocol                                                                                                                                                  
ansible.builtin.ssh            connect via SSH client binary                                                                                                                                                                          
ansible.builtin.winrm          Run tasks over Microsoft's WinRM                                                                                                                                                                       
ansible.netcommon.grpc         Provides a persistent connection using the gRPC protocol                                                                                                                                               
ansible.netcommon.httpapi      Use httpapi to run command on network appliances                                                                                                                                                       
ansible.netcommon.libssh       Run tasks using libssh for ssh connection                                                                                                                                                              
ansible.netcommon.netconf      Provides a persistent connection using the netconf protocol                                                                                                                                            
ansible.netcommon.network_cli  Use network_cli to run command on network appliances                                                                                                                                                   
ansible.netcommon.persistent   Use a persistent unix socket for connection                                                                                                                                                            
community.aws.aws_ssm          execute via AWS Systems Manager                                                                                                                                                                        
community.docker.docker        Run tasks in docker containers                                                                                                                                                                         
community.docker.docker_api    Run tasks in docker containers                                                                                                                                                                         
community.docker.nsenter       execute on host running controller container                                                                                                                                                           
community.general.chroot       Interact with local chroot                                                                                                                                                                             
community.general.funcd        Use funcd to connect to target                                                                                                                                                                         
community.general.iocage       Run tasks in iocage jails                                                                                                                                                                              
community.general.jail         Run tasks in jails                                                                                                                                                                                     
community.general.lxc          Run tasks in lxc containers via lxc python library                                                                                                                                                     
community.general.lxd          Run tasks in lxc containers via lxc CLI                                                                                                                                                                
community.general.qubes        Interact with an existing QubesOS AppVM                                                                                                                                                                
community.general.saltstack    Allow ansible to piggyback on salt minions                                                                                                                                                             
community.general.zone         Run tasks in a zone instance                                                                                                                                                                           
community.libvirt.libvirt_lxc  Run tasks in lxc containers via libvirt                                                                                                                                                                
community.libvirt.libvirt_qemu Run tasks on libvirt/qemu virtual machines                                                                                                                                                             
community.okd.oc               Execute tasks in pods running on OpenShift                                                                                                                                                             
community.vmware.vmware_tools  Execute tasks inside a VM via VMware Tools                                                                                                                                                             
containers.podman.buildah      Interact with an existing buildah container                                                                                                                                                            
containers.podman.podman       Interact with an existing podman container                                                                                                                                                             
kubernetes.core.kubectl        Execute tasks in pods running on Kubernetes   



ansible.builtin.local          execute on controller
```
10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.
Ответ:
```
root@FEDORA /h/v/N/8/playbook# cat inventory/prod.yml 
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker
  local:
    hosts:
      localhost:
        ansible_connection: local

```
11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь, что факты `some_fact` для каждого из хостов определены из верных `group_vars`.
Ответ:
```
root@FEDORA /h/v/N/8/playbook# ansible-playbook site.yml -i inventory/prod.yml

PLAY [Print os facts] *********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************************************
ok: [localhost]
ok: [centos7]
ok: [ubuntu]

TASK [Print OS] ***************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [localhost] => {
    "msg": "Fedora"
}

TASK [Print fact] *************************************************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP ********************************************************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```
12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.
Ответ:
```
Готово. Запушил в репозитарий
```

## Необязательная часть

1. При помощи `ansible-vault` расшифруйте все зашифрованные файлы с переменными.
2. Зашифруйте отдельное значение `PaSSw0rd` для переменной `some_fact` паролем `netology`. Добавьте полученное значение в `group_vars/all/exmp.yml`.
3. Запустите `playbook`, убедитесь, что для нужных хостов применился новый `fact`.
4. Добавьте новую группу хостов `fedora`, самостоятельно придумайте для неё переменную. В качестве образа можно использовать [этот вариант](https://hub.docker.com/r/pycontribs/fedora).
5. Напишите скрипт на bash: автоматизируйте поднятие необходимых контейнеров, запуск ansible-playbook и остановку контейнеров.
6. Все изменения должны быть зафиксированы и отправлены в ваш личный репозиторий.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
