FROM 10.0.88.85:5000/ubuntu:22.04 
RUN apt-get update ; apt-get -y install fortune 
ADD fortuneloop-args.sh /bin/fortuneloop.sh
RUN chmod +x /bin/fortuneloop.sh
ENTRYPOINT ["/bin/fortuneloop.sh"]