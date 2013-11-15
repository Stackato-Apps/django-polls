web: uwsgi --reaper --vacuum --master --processes 8 --single-interpreter --enable-threads --http-socket=0.0.0.0:$PORT --static-map /static=$HOME/static mysite/wsgi.py
test: python manage.py test
