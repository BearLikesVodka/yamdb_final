FROM python:3.7-slim

LABEL author='bearlikesvodka@gmail.com' version='1.0'

WORKDIR /.

COPY . .

RUN pip3 install -r requirements.txt --no-cache-dir

CMD ["gunicorn", "api_yamdb.wsgi:application", "--bind", "0:8000" ]
