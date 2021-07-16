FROM 050665733583.dkr.ecr.eu-west-1.amazonaws.com/airflow-base:master


ENV AIRFLOW__CORE__FERNET_KEY="3BF2HkbYhQgmRjRo6_3AxVMZyfmIHq_NWOYOVVol64k="
COPY docker/airflow.cfg ${AIRFLOW_HOME}/airflow.cfg
COPY dags /usr/local/airflow/dags

USER root
RUN chmod 777 /entrypoint.sh

USER airflow
COPY docker/requirements/prod.txt $AIRFLOW_HOME/prod.txt
COPY docker/requirements/base.txt $AIRFLOW_HOME/base.txt

# Install the requirements file
RUN pip install -r prod.txt
