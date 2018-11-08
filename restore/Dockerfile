FROM docker.io/alaypatel07/ansible-operator
USER root
RUN pip install etcd3
RUN pip install boto3
RUN pip install botocore

RUN useradd -u ${USER_UID} ${USER_NAME}
USER ${USER_NAME}

COPY get_file.yaml /tmp/ansible/get_file.yaml
