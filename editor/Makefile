protobuf = protobuf-3.12.0

protoc_version = $(shell protoc --version | sed 's/.* //')
protobuf_version = $(shell echo $(protobuf) | sed 's/.*-//')

closure = closure-library-20200517

all : prep py js

prep :
	# Ensure output directories exist.
	mkdir -p gen/py gen/js/ord gen/js/proto/ord
	# Prevent Closure from compiling test code.
	rm -rf $(protobuf)/js/binary/*test* $(protobuf)/js/*test* $(protobuf)/js/experimental
	# Ensure the proto compiler matches the proto runtime JS.
	if [ $(protoc_version) != $(protobuf_version) ]; then false; fi

js : gen/js/proto/ord/dataset.js \
	   gen/js/proto/ord/reaction.js \
	   gen/js/ord/dataset.js \
	   gen/js/ord/reaction.js \

py : gen/py/ord_schema/proto/dataset_pb2.py \
	   gen/py/ord_schema/proto/reaction_pb2.py

gen/js/ord/dataset.js : js/dataset.js gen/js/proto/ord/dataset.js
	$(closure)/closure/bin/build/closurebuilder.py \
	  --root=$(closure) \
	  --root=$(protobuf)/js \
	  --root=gen/js/proto/ord \
	  --root=js \
	  --namespace=ord.dataset \
	  --output_mode=script \
	  --output_file=gen/js/ord/dataset.js \
	  || (rm -f gen/js/ord/dataset.js && false)

gen/js/ord/reaction.js : js/*.js gen/js/proto/ord/reaction.js
	$(closure)/closure/bin/build/closurebuilder.py \
	  --root=$(closure) \
	  --root=$(protobuf)/js \
	  --root=gen/js/proto/ord \
	  --root=js \
	  --namespace=ord.reaction \
	  --output_mode=script \
	  --output_file=gen/js/ord/reaction.js \
	  || (rm -f gen/js/ord/reaction.js && false)

gen/js/proto/ord/%.js : ../../ord-schema/proto/%.proto
	protoc \
	  --experimental_allow_proto3_optional \
	  --js_out=binary:gen/js/proto/ord \
	  --proto_path=../.. \
	  $<

gen/py/ord_schema/proto/%_pb2.py : ../../ord-schema/proto/%.proto
	protoc \
	  --experimental_allow_proto3_optional \
	  --python_out=gen/py \
	  --proto_path=../.. \
	$<

package : all
	rm -rf package
	mkdir -p package/ord

	cp -r \
		css \
	  gen \
	  html \
	  py \
	  serve.sh \
	  package/ord

	tar --create --directory=package --gzip --file=package/ord.tgz ord

test : all
	node js/test.js

clean :
	rm -rf gen
	rm -rf package