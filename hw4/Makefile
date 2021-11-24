PWD = $(shell pwd)

deploy-fault:
	kubectl apply -f ./k8s/vs-fault.yaml

deploy-delay:
	kubectl apply -f ./k8s/vs-delay.yaml

deploy-fflag:
	kubectl apply -f ./k8s/depl.yaml
	kubectl apply -f ./k8s/dr.yaml
	kubectl apply -f ./k8s/vs.yaml
	kubectl apply -f ./k8s/ef.yaml