APP_ID:=__APP_ID__
VERSION:=v$(shell date +%YM%mD%dH%HM%M)
IMG=__IMG_PREFIX__/$(APP_ID):$(VERSION)

define prompt-for
	@{ \
	set -e ;\
	read -p $(1) value ; \
	echo $(3) | xargs -n 1 sed -i .tmp 's|__$(2)__|'$$value'|' ; \
	rm *.tmp
	}
endef

release: .staging build
	gcloud docker push $(IMG)
	kubectl patch deployment $(NAME)-staging -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "nginx", "image": "$(IMG)" } ] } } } }'
	sed -i .tmp 's|image:.*|image: "$(IMG)"|' staging.yaml
	rm staging.yaml.tmp
	echo $(IMG) > .staging

approve-staging: .production
	kubectl patch deployment $(NAME)-production -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "nginx", "image": "$(shell cat .staging)" } ] } } } }'
	sed -i .tmp 's|image:.*|image: "$(shell cat .staging)"|' production.yaml
	rm production.yaml.tmp
	cp .staging .production

release-production: .production build
	gcloud docker push $(IMG)
	kubectl patch deployment $(NAME)-production -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "nginx", "image": "$(IMG)" } ] } } } }'
	sed -i .tmp 's|image:.*|image: "$(IMG)"|' production.yaml
	rm production.yaml.tmp
	echo $(IMG) > .production

build:
	cd .. && make build
	docker build -t $(IMG)

.staging: .prepared staging.yaml
	kubectl apply -f staging.yaml
	touch .staging

.production: .prepared production.yaml
	kubectl apply -f production.yaml
	touch .production

.prepared:
	@echo Configure your app...
	$(call prompt-for,"Repo prefix (e.g. gcr.io/your-cluster or quay.io/yourorg/) : ",IMG_PREFIX,Makefile)
	$(call prompt-for,"App identifier: ",APP_ID,Makefile production.yaml staging.yaml)
	$(call prompt-for,'Production virtual host: ',VIRTUAL_HOST,production.yaml)
	$(call prompt-for,'Staging virtual host: ',STAGING_VIRTUAL_HOST,staging.yaml)
	touch .prepared
	@echo You are now configured - re-run your previous command.
	exit -1
