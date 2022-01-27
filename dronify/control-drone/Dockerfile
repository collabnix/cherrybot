FROM python


RUN apt update
RUN /usr/local/bin/python -m pip install --upgrade pip 
RUN apt install -y python3-dev python3-pip

COPY requirements.txt /opt/requirements.txt
COPY drone-control.py /opt/drone-control.py
RUN pip3 install -r /opt/requirements.txt


CMD [ "python3", "/opt/drone-control.py"]

