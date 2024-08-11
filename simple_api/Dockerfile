FROM python:3.8-buster
LABEL desc="This is mini-project on Docker : POZOS/Student-list"
MAINTAINER john (john.vieira.pro@gmail.com)
RUN apt-get update -y && apt-get install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
RUN pip install --upgrade pip
RUN pip3 install flask==2.0.0 flask_httpauth==4.1.0 flask_simpleldap python-dotenv==0.14.0 Werkzeug==2.0.0
COPY ./student_age.py /
EXPOSE 5000
RUN mkdir /data
VOLUME /data
COPY ./student_age.json /data/
CMD [ "python3", "./student_age.py" ]