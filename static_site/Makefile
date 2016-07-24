VERSION:=v$(shell date +%YM%mD%dH%HM%M)
IMG=__IMG_PREFIX__/__APP_ID__:$(VERSION)

define prompt-for
	@{ \
	set -e ;\
	read -p $(1) value ; \
	echo $(3) | xargs -n 1 sed -i .tmp 's|$(2)|'$$value'|' ; \
	}
endef

release: .prepared
	docker build -t $(IMG) .
	gcloud docker push $(IMG)
	kubectl patch deployment thehumaneffort-com -p '{ "spec": { "template": { "spec": { "containers": [ { "name": "nginx", "image": "$(IMG)" } ] } } } }'

.prepared:
	@echo Configure your app...
	$(call prompt-for,"Repo prefix (e.g. gcr.io/your-cluster or quay.io/yourorg/) : ",__IMG_PREFIX__,Makefile)
	$(call prompt-for,"App identifier: ",__APP_ID__,Makefile production.yaml staging.yaml)
	$(call prompt-for,'Production virtual host: ',__VIRTUAL_HOST__,production.yaml)
	$(call prompt-for,'Staging virtual host: ',__STAGING_VIRTUAL_HOST__,staging.yaml)
	touch .prepared
	exit -1
