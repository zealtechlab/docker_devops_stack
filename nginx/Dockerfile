FROM nginx:alpine
LABEL Prabhakaran Sampath <prabhakaran@zealtechlab.co.in>
COPY nginx.conf /etc/nginx/nginx.conf
COPY ./site /var/www/html
RUN apk --no-cache add curl