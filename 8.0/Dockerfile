FROM ubuntu:focal as builder

ARG MYSQL_VERSION=8.0.33

WORKDIR /root
COPY ./sources.list /etc/apt/sources.list

RUN cat /etc/apt/sources.list
RUN rm -Rf /var/lib/apt/lists/*
RUN ln -fs /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
RUN DEBIAN_FRONTEND=noninteractive apt update -y
RUN DEBIAN_FRONTEND=noninteractive apt install cmake make gcc g++ openssl bison pkg-config -y
RUN DEBIAN_FRONTEND=noninteractive apt install build-essential wget curl libncurses5-dev libssl-dev -y -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold'

#    wget https://cdn.mysql.com/Downloads/MySQL-8.0/mysql-boost-${MYSQL_VERSION}.tar.gz && \

COPY mysql-boost-${MYSQL_VERSION}.tar.gz ./mysql-boost-${MYSQL_VERSION}.tar.gz
RUN tar zxf ./mysql-boost-${MYSQL_VERSION}.tar.gz && \
    ls -la 
      

RUN export EXTRA_CFLAGS="-O0 -g" && cd mysql-${MYSQL_VERSION} && cmake ./ -DWITH_DEBUG=1 -DFORCE_INSOURCE_BUILD=1 -DWITH_INNODB_EXTRA_DEBUG=1 -DCMAKE_INSTALL_PREFIX=/mysql-debug -DDOWNLOAD_BOOST=1 -DWITH_BOOST=./boost && \
    make -j$(getconf _NPROCESSORS_ONLN) install && \
    cd .. 
#RUN rm -rf mysql-${MYSQL_VERSION}
WORKDIR /mysql-debug
RUN ./bin/mysqld  --no-defaults --datadir=./data --basedir=./ --initialize-insecure

FROM ubuntu:focal
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install gdb gdbserver -y -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' && \
    echo '[mysqld]\nskip-host-cache\nskip-name-resolve\nskip-grant-tables' > /etc/my.cnf && \
    rm /etc/sysctl.d/10-ptrace.conf && \
    sysctl -p

COPY --from=builder /mysql-debug /mysql-debug
EXPOSE 3306
EXPOSE 2345
CMD cd /mysql-debug; sudo gdbserver :2345 ./bin/mysqld --user=root --skip-grant-tables --datadir=./data --basedir=./ --user=root
