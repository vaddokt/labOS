FROM debian
ADD debian.list /etc/apt/sources.list.d/
RUN apt-get update && apt-get -y upgrade && apt -y install nginx && apt-get clean && \
	cd /var/www/ && rm -rf ./* && \
	mkdir -p vaddokt.com/img && \
	chmod -R 754 /var/www/vaddokt.com/ && \
	useradd Vadim && groupadd Doktorov && usermod -aG Doktorov Vadim && \
	chown -R Vadim:Doktorov /var/www/vaddokt.com/ && \
	sed -i 's/\/var\/www\/html/\/var\/www\/vaddokt.com/g' /etc/nginx/sites-enabled/default && \
	sed -i 's/user www-data/user Vadim/g' /etc/nginx/nginx.conf
ADD index.html /var/www/vaddokt.com/
ADD img.jpg /var/www/vaddokt.com/img/
CMD ["nginx", "-g", "daemon off;"]
