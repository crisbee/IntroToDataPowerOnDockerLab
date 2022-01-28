FROM ibmcom/datapower:2018.4.1

ENV  DATAPOWER_ACCEPT_LICENSE=true      DATAPOWER_INTERACTIVE=true      DATAPOWER_WORKER_THREADS=2 
EXPOSE 9090

ADD ./config /drouter/config
ADD ./local /drouter/local
