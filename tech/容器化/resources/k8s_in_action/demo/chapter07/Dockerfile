FROM 192.168.5.5:5000/ubuntu:24.10 
RUN apt-get update ; apt-get -y install fortune 
ADD fortuneloop-args.sh /bin/fortuneloop.sh
RUN chmod +x /bin/fortuneloop.sh
ENTRYPOINT ["/bin/fortuneloop.sh"]