FROM nginx:alpine
 
COPY nginx.conf /etc/nginx/nginx.conf
COPY ./site /var/www/html
RUN apk --no-cache add curl