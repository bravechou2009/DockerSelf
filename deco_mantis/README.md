# Introduce
# This dockerfile is suitable for installing MantisBT

## How to run
Change prompt to this folder and then execute
```sh
/usr/local/bin/docker-compose up -d
```
If you just only want to run the maintisBT separately, you can run use following command:
```sh
docker run  -d --privileged -P --name zhouyong_deco_maintsBT -h hucaeah0.cn.hpecorp.net  -v /sys/fs/cgroup:/sys/fs/cgroup:ro  hucaeah0.cn.hpecorp.net:16688/deco_mantis:V1.0 /bin/bash
```
## How to build images, you can build with make command
```
	1. make build        - build the deco mantis image
	2. make maintis      - start maintis separately
	3. make stop         - stop maintis
	4. make logs         - view logs
	5. make purge        - stop and remove the container
	6. make compose      - start maintis compose
```
## How to access maintisBT
Access http://Docker Host IP:8989/mantisbt url