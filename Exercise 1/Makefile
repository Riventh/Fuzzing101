build:
	docker build -t fuzzing101:exercise1 .

run: build
	mkdir -p out
	docker run --rm -it -v ./out:/root/fuzzing_xpdf/out/ fuzzing101:exercise1

