FROM arm64v8/python:2-alpine3.10

COPY ./pgadmin4-4.9-py2.py3-none-any.whl /opt/pgadmin4-4.9-py2.py3-none-any.whl
COPY ./pgadmin4-4.9-py2.py3-none-any.whl.asc /opt/pgadmin4-4.9-py2.py3-none-any.whl.asc

# create a non-privileged user to use at runtime
RUN echo 'http://mirrors.ustc.edu.cn/alpine/v3.10/main' > /etc/apk/repositories \
    && echo 'http://mirrors.ustc.edu.cn/alpine/v3.10/community' >>/etc/apk/repositories \
    && mkdir -p /~/.pip && touch /~/.pip/pip.conf \
    && echo -e "[global] \n index-url = https://mirrors.aliyun.com/pypi/simple/ \n [install] \n trusted-host=mirrors.aliyun.com" > /~/.pip/pip.conf \
    && apk add -U tzdata \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && apk del tzdata \
    && set -x \
    && addgroup -g 50 -S pgadmin \
    && adduser -D -S -h /pgadmin -s /sbin/nologin -u 1000 -G pgadmin pgadmin \
    && mkdir -p /pgadmin/config /pgadmin/storage \
    && chown -R 1000:50 /pgadmin

# Install postgresql tools for backup/restore
RUN apk add --no-cache libedit postgresql \
 && cp /usr/bin/psql /usr/bin/pg_dump /usr/bin/pg_dumpall /usr/bin/pg_restore /usr/local/bin/ \
 && apk del postgresql

RUN apk add --no-cache postgresql-dev libffi-dev

ENV PGADMIN_VERSION=4.9
ENV PYTHONDONTWRITEBYTECODE=1

RUN apk add --no-cache alpine-sdk linux-headers \
    && python -m pip install --upgrade --force pip \
    && pip install setuptools \
    # https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v4.9/pip/
	# git clone --branch $PGADMIN4_TAG --depth 1 https://git.postgresql.org/git/pgadmin4.git
    && echo "file:///opt/pgadmin4-${PGADMIN_VERSION}-py2.py3-none-any.whl" | pip install --no-cache-dir -i https://pypi.tuna.tsinghua.edu.cn/simple -r /dev/stdin \
    && apk del alpine-sdk linux-headers \
    && rm /opt/pgadmin4-${PGADMIN_VERSION}-py2.py3-none-any.whl \
    && apk del build-dependencies

EXPOSE 5050

COPY LICENSE config_distro.py /usr/local/lib/python2.7/site-packages/pgadmin4/

USER pgadmin:pgadmin
CMD ["python", "./usr/local/lib/python2.7/site-packages/pgadmin4/pgAdmin4.py"]
# CMD ["sh", "-c", "python ${PACKAGE_DIR}/pgAdmin4.py"]
VOLUME /pgadmin/

## ****************************** 参考资料 *****************************************
## 制作Docker Image: docker build --no-cache -t idu/pgadmin4-alpine:arm64v8 .
## docker run \
## -d \
## -p 6060:5050 \
## --net=idu-qb-saas-network  \
## --ip=192.168.200.25 \
## --hostname=idu-qb-pgadmin \
## --name idu-qb-pgadmin \
## --read-only idu/pgadmin4-alpine:arm64v8
