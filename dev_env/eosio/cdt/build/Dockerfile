FROM eosio/eos-dev

RUN apt update

RUN apt upgrade -y

RUN apt install -y wget apt-utils

RUN wget https://github.com/EOSIO/eosio.cdt/releases/download/v1.6.1/eosio.cdt_1.6.1-1_amd64.deb

RUN apt install ./eosio.cdt_1.6.1-1_amd64.deb

RUN rm -rf ./eosio.cdt_1.6.1-1_amd64.deb
