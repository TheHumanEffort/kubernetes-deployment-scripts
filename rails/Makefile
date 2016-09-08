NAME:=__APP_ID__
VERSION:=v$(shell date +%YM%mD%dH%HM%M)
REPO=__IMG_PREFIX__/$(NAME)
IMG=$(REPO):$(VERSION)

POSTGRES_SECRET_NAME=$(NAME)-pg-auth
POSTGRES_DISK_NAME=$(NAME)-pg-disk
POSTGRES_DISK_SIZE=20
ZONE=us-central1-c

define prompt-for
	@{ \
	set -e ;\
	read -p $(1) value ; \
	echo $(3) | xargs -n 1 sed -i .tmp 's|__$(2)__|'$$value'|' ; \
	rm *.tmp ;\
	}
endef

release: .staging build release-staging
release-patch: .staging patch-build release-staging

release-staging:
	docker push $(IMG)
	kubectl patch deployment $(NAME)-staging -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "rails", "image": "$(IMG)" } ] } } } }'
	sed -i .tmp 's|image:.*|image: "$(IMG)"|' staging.yaml
	rm staging.yaml.tmp
	echo $(IMG) > .staging
	@echo 'NOTICE!!!!!!'
	@echo 'You probably have to initialize your database!!!!'
	@echo 'You can do this with some variation of:'
	@echo 'kubectl exec -it $$(kubectl get pod | grep $(NAME)-staging | awk '\''{ print $$1 }'\'' ) rake db:migrate'
	@echo "Once this is done, you can delete this notice, remove the tail -f /dev/null from staging.yaml and `make .staging`"

approve-staging: .production
	kubectl patch deployment $(NAME)-production -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "rails", "image": "$(shell cat .staging)" } ] } } } }'
	sed -i .tmp 's|image:.*|image: "$(shell cat .staging)"|' production.yaml
	rm production.yaml.tmp
	cp .staging .production
	@echo 'NOTICE!!!!!!'
	@echo 'You probably have to initialize your database!!!!'
	@echo 'You can do this with some variation of:'
	@echo 'kubectl exec -it $$(kubectl get pod | grep $(NAME)-production | awk '\''{ print $$1 }'\'' ) rake db:migrate'
	@echo "Once this is done, you can delete this notice, remove the tail -f /dev/null from production.yaml and `make .production`"

release-production: .production build
	docker push $(IMG)
	kubectl patch deployment $(NAME)-production -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "rails", "image": "$(IMG)" } ] } } } }'
	sed -i .tmp 's|image:.*|image: "$(IMG)"|' production.yaml
	rm production.yaml.tmp
	echo $(IMG) > .production
	@echo 'NOTICE!!!!!!'
	@echo 'You probably have to initialize your database!!!!'
	@echo 'You can do this with some variation of:'
	@echo 'kubectl exec -it $$(kubectl get pod | grep $(NAME)-production | awk '\''{ print $$1 }'\'' ) rake db:migrate'
	@echo "Once this is done, you can delete this notice, remove the tail -f /dev/null from production.yaml and `make .production`"

patch-build: FORCE
	docker rm -f patch-latest || echo "nothing to delete"
	docker run --name patch-latest $(REPO):latest bash -c "find /application -type f -exec stat --format '%Y :%y\!%n' \"{}\" \; | sort -nr | head -n 1" | awk -F '!' '{ print $$2 }'> last_file
	bash -c "docker cp patch-latest:$$(cat last_file) .last-build && rm last_file"
	cd .. && find app db config lib -type f -newer .last-build | awk '{ print "echo cp "$$1" ; docker cp "$$1" patch-latest:/application/"$$1  }' | bash
	rm .last-build
	docker commit patch-latest $(REPO):$(VERSION)

build:
	docker build -t $(REPO):latest ..
	docker tag $(REPO):latest $(IMG)

.staging: .prepared .postgres staging.yaml
	kubectl apply -f staging.yaml
	touch .staging

.production: .prepared .postgres .secret_key_base production.yaml
	kubectl apply -f production.yaml
	touch .production

.secret_key_base:
	kubectl create secret generic $(NAME)-secret-key --from-literal=key-base=p$(shell openssl rand -hex 64)
	touch .secret_key_base

.postgres: postgres.yaml .pgsecret .pgdisk
	kubectl apply -f postgres.yaml
	touch .postgres

.pgsecret:
	kubectl create secret generic $(POSTGRES_SECRET_NAME) --from-literal=username=$(NAME)-postgres --from-literal=password=p$(shell openssl rand -hex 32)
	touch .pgsecret

.pgdisk:
# example for google cloud:
	gcloud compute disks create $(POSTGRES_DISK_NAME) --size $(POSTGRES_DISK_SIZE) --type pd-ssd --zone $(ZONE)
	touch .pgdisk

.prepared:
	@echo Configure your app...
	$(call prompt-for,"Repo prefix (e.g. gcr.io/your-cluster or quay.io/yourorg/) : ",IMG_PREFIX,Makefile)
	$(call prompt-for,"App identifier: ",APP_ID,Makefile production.yaml staging.yaml postgres.yaml)
	$(call prompt-for,'Production virtual host: ',VIRTUAL_HOST,production.yaml)
	$(call prompt-for,'Staging virtual host: ',STAGING_VIRTUAL_HOST,staging.yaml)
	touch .prepared
	@echo You are now configured - re-run your previous command.
	exit -1
