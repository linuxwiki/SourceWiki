# Django

uwsgi 配置模版:

    [uwsgi]
    procname-prefix = xxxxx                      # 进程前缀，方便 grep 区分
    chdir           = /data/${proj_name}
    wsgi-file       = ${proj_name}/wsgi.py
    socket          = 127.0.0.1:${port}
    master          = true
    workers         = 2
    procname-master = uwsgi master
    procname        = uwsgi worker
    vacuum          = true
    daemonize2      = /data/log/uwsgi/${proj_name}.log
    pidfile         = /var/run/${proj_name}.pid
