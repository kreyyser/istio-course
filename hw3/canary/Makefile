PWD = $(shell pwd)

## CANARY RELATED SCRIPTS

deploy-canary-v2-10: deploy-v2-depl deploy-v2-dr deploy-authors-vs-10 deploy-books-vs-10

deploy-canary-v2-50: deploy-v2-depl deploy-v2-dr deploy-authors-vs-50 deploy-books-vs-50

deploy-canary-v2-100: deploy-v2-depl deploy-v2-dr deploy-authors-vs-100 deploy-books-vs-100

deploy-v2-depl: deploy-authors-v2 deploy-books-v2

deploy-v2-dr: deploy-authors-dr deploy-books-dr



## AUTHOR RELATED SCRIPTS
deploy-authors-v2:
	kubectl apply -f ./author/deployment.yaml

deploy-authors-dr:
	kubectl apply -f ./author/dr.yaml

deploy-authors-vs-10:
	kubectl apply -f ./author/vs-10.yaml

deploy-authors-vs-50:
	kubectl apply -f ./author/vs-50.yaml

deploy-authors-vs-100:
	kubectl apply -f ./author/vs-100.yaml



## BOOK RELATED SCRIPTS
deploy-books-v2:
	kubectl apply -f ./book/deployment.yaml

deploy-books-dr:
	kubectl apply -f ./book/dr.yaml

deploy-books-vs-10:
	kubectl apply -f ./book/vs-10.yaml

deploy-books-vs-50:
	kubectl apply -f ./book/vs-50.yaml

deploy-books-vs-100:
	kubectl apply -f ./book/vs-100.yaml
