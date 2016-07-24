NAME:=__APP_ID__
VERSION:=v$(shell date +%YM%mD%dH%HM%M)
IMG=__IMG_PREFIX__/$(NAME):$(VERSION)

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

release: .staging build
	gcloud docker push $(IMG)
	kubectl patch deployment $(NAME)-staging -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "rails", "image": "$(IMG)" } ] } } } }'
	sed -i .tmp 's|image:.*|image: "$(IMG)"|' staging.yaml
	rm staging.yaml.tmp
	echo $(IMG) > .staging

approve-staging: .production
	kubectl patch deployment $(NAME)-production -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "rails", "image": "$(shell cat .staging)" } ] } } } }'
	sed -i .tmp 's|image:.*|image: "$(shell cat .staging)"|' production.yaml
	rm production.yaml.tmp
	cp .staging .production

release-production: .production build
	gcloud docker push $(IMG)
	kubectl patch deployment $(NAME)-production -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "rails", "image": "$(IMG)" } ] } } } }'
	sed -i .tmp 's|image:.*|image: "$(IMG)"|' production.yaml
	rm production.yaml.tmp
	echo $(IMG) > .production

build:
	docker build -t $(IMG) ..

.staging: .prepared staging.yaml
	kubectl apply -f staging.yaml
	touch .staging

.production: .prepared production.yaml
	kubectl apply -f production.yaml
	touch .production

.postgres: postgres.yaml .pgsecret .pgdisk
	kubectl apply -f postgres.yaml
	touch .postgres

.pgsecret:
	kubectl create secret generic $(POSTGRES_SECRET_NAME) --from-literal=username=$(NAME)-postgres --from-literal=password=p$(shell openssl rand -hex 32)
	touch .pgsecret

.pgdisk:
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
