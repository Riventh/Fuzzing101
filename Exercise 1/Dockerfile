FROM ubuntu

RUN apt update && apt upgrade -y
RUN apt install -y \
    wget \
    sudo \
    python3-pip

WORKDIR /root/
RUN mkdir fuzzing_xpdf
WORKDIR /root/fuzzing_xpdf 

RUN sudo apt install -y build-essential
RUN wget https://dl.xpdfreader.com/old/xpdf-3.02.tar.gz
RUN tar -xvzf xpdf-3.02.tar.gz

WORKDIR /root/fuzzing_xpdf/xpdf-3.02
RUN sudo apt update && sudo apt install -y build-essential gcc 
RUN ./configure --prefix="$HOME/fuzzing_xpdf/install/"
RUN make
RUN make install

WORKDIR /root/fuzzing_xpdf
RUN mkdir pdf_examples
WORKDIR /root/fuzzing_xpdf/pdf_examples
RUN wget https://github.com/mozilla/pdf.js-sample-files/raw/master/helloworld.pdf
RUN wget https://www.melbpc.org.au/wp-content/uploads/2017/10/small-example-pdf-file.pdf

RUN /root/fuzzing_xpdf/install/bin/pdfinfo -box -meta /root/fuzzing_xpdf/pdf_examples/helloworld.pdf

RUN sudo apt-get update
RUN sudo apt-get install -y build-essential python3-dev automake git flex bison libglib2.0-dev libpixman-1-dev python3-setuptools
RUN sudo apt-get install -y lld-11 llvm-11 llvm-11-dev clang-11 || sudo apt-get install -y lld llvm llvm-dev clang
RUN sudo apt-get install -y gcc-$(gcc --version|head -n1|sed 's/.* //'|sed 's/\..*//')-plugin-dev libstdc++-$(gcc --version|head -n1|sed 's/.* //'|sed 's/\..*//')-dev

WORKDIR /root
RUN git clone https://github.com/AFLplusplus/AFLplusplus
WORKDIR AFLplusplus
# Switch to old version (compatibility) of AFL++
RUN git checkout 3.11c
RUN git switch -c 3.11c
RUN export LLVM_CONFIG="llvm-config-11"
RUN make distrib
RUN sudo make install

RUN rm -r /root/fuzzing_xpdf/install
WORKDIR /root/fuzzing_xpdf/xpdf-3.02/
RUN make clean

# Recompile using AFL++
RUN export LLVM_CONFIG="llvm-config-11"
RUN CC=$HOME/AFLplusplus/afl-clang-fast CXX=$HOME/AFLplusplus/afl-clang-fast++ ./configure --prefix="$HOME/fuzzing_xpdf/install/"
RUN make
RUN make install

# Fuzz
CMD afl-fuzz -i $HOME/fuzzing_xpdf/pdf_examples/ -o $HOME/fuzzing_xpdf/out/ -s 123 -- $HOME/fuzzing_xpdf/install/bin/pdftotext @@ $HOME/fuzzing_xpdf/output
