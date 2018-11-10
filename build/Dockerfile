FROM docker.io/alaypatel07/ansible-operator:latest
USER root
RUN pip install etcd3
RUN pip install boto3
RUN pip install botocore

RUN useradd -u ${USER_UID} ${USER_NAME}
USER ${USER_NAME}

ENV RESYNC_PERIOD=60
COPY ansible/roles/ ${HOME}/roles/
COPY ansible/playbook.yaml ${HOME}/playbook.yaml
COPY ansible/backup_playbook.yaml ${HOME}/backup_playbook.yaml
COPY ansible/restore_playbook.yaml ${HOME}/restore_playbook.yaml
COPY watches.yaml ${HOME}/watches.yaml
