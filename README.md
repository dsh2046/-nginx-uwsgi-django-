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

6.测试 uwsgi (进入Django项目子目录)
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
server_name 你的nginx运行的本机ip地址 ; # substitute your machine's IP address or FQDN
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

8.将该配置文件加入到nginx的启动配置文件中
  sudo ln -s 你的目录/Mxonline/conf/nginx/uc_nginx.conf /etc/nginx/conf.d/
  
9.在django的setting文件中，添加下面一行内容：
　STATIC_ROOT = os.path.join(BASE_DIR, "static/")
　运行命令
　python manage.py collectstatic

10.运行nginx
sudo /usr/sbin/nginx
这里需要注意 一定是直接用nginx命令启动， 不要用systemctl启动nginx不然会有权限问题

11.通过配置文件启动uwsgi
新建uwsgi.ini 配置文件， 内容如下：

# mysite_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = 你的Django目录
# Django's wsgi file
module          = MxOnline.wsgi
# the virtualenv (full path)

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 10
# the socket (use the full path to be safe
socket          = 127.0.0.1:8000
# ... with appropriate permissions - may be needed
# chmod-socket    = 664
# clear environment on exit
vacuum          = true
virtualenv = 你的virtualenv目录


12.启动uWSGI
uwsgi -i 你的目录/uwsgi.ini

