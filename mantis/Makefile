all: help

help:
	@echo ""
	@echo "-- Help Menu"
	@echo ""
	@echo "   1. make build        - build the deco mantis image"
	@echo "   2. make maintis	   - start maintis separately"
	@echo "   3. make stop         - stop maintis"
	@echo "   4. make logs         - view logs"
	@echo "   5. make purge        - stop and remove the container"
	@echo "   6. make compose 	   - start maintis compose"

build:
	@docker build -t hucaeah0.cn.hpecorp.net:16688/deco_mantis:V1.0 .
	
maintis:
	@echo "Starting maintis..."
	@docker run  -d --privileged -P \
		--name='deco_maintsBT' \
		-h hucaeah0.cn.hpecorp.net \
		-v /sys/fs/cgroup:/sys/fs/cgroup:ro \
		hucaeah0.cn.hpecorp.net:16688/deco_mantis:V1.0  \
		/bin/bash >/dev/null
	@echo "Please be patient. This could take a while..."
	@echo "Maintis will be available at http://localhost:PORT"
	@echo "Type 'make logs' for the logs"

stop:
	@echo "Stopping maintis..."
	@docker stop mantisbt_portal >/dev/null
	@docker stop mantisbt_db >/dev/null
	@docker rm mantisbt_portal >/dev/null
	@docker rm mantisbt_db >/dev/null

logs:
	@docker logs -f mantisbt_portal
	
compose:
	/usr/local/bin/docker-compose up -d	
	
	