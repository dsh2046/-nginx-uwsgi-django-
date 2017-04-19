1.安装 nginx
  sudo apt-get install nginx

2.安装 virtualenv
  pip install virtualenv

3.创建并进入虚拟环境
  virtualenv venv   
  source venv/bin/activate

4.通过 pip freeze > requirements.txt 将本地的虚拟环境安装包相信信息导出来
  然后将requirements.txt文件上传到服务器之后运行：
  pip install -r requirements.txt   安装依赖包
  
5.安装 uwsgi
  pip install uwsgi

6.测试 uwsgi
  uwsgi --http :8000 --module MxOnline.wsgi
  
7.配置 nginx
新建uc_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
# server unix:///path/to/your/mysite/mysite.sock; # for a file socket
server 127.0.0.1:8000; # for a web port socket (we'll use this first)
}
# configuration of the server

server {
# the port your site will be served on
listen      80;
# the domain name it will serve for
server_name 你的ip地址 ; # substitute your machine's IP address or FQDN
charset     utf-8;

# max upload size
client_max_body_size 75M;   # adjust to taste

# Django media
location /media  {
    alias 你的目录/Mxonline/media;  # 指向django的media目录
}

location /static {
    alias 你的目录/Mxonline/static; # 指向django的static目录
}

# Finally, send all non-media requests to the Django server.
location / {
    uwsgi_pass  django;
    include     uwsgi_params; # the uwsgi_params file you installed
}
}
