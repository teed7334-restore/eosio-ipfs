# eosio-ipfs
EOSIO + IPFS 私鏈快速生成器

## 資料夾結構

## 必須套件
本系統透過Docker與Docker-Compose生成環境，請先設置好EOSIO + IPFS

## EOSIO 私鏈架設流程
1. 進入容器
```
docker exec -it nodeos bash
```

2. 新增錢包
```
cleos wallet create --file ~/wallet.passwd
```

3. 開啟錢包
```
cleos wallet open
```

4. 查看目前己開啟錢包
```
cleos wallet list
```

5. 解鎖錢包，密碼在~/wallet.passwd檔案中
```
cleos wallet unlock
```

6. 匯入設定檔中私鑰，私鑰在/mnt/dev/config/config.ini中，裡面的signature-provider，例如我這邊私鑰是5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```
cleos wallet import --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

7. 新增二把含以上的公鑰，等等新增會員時要用到
```
cleos wallet create_key
```

8. 查看可用的公私鑰
```
cleos wallet private_keys
```

9. 新增會員帳號
```
cleos create account eosio [會員帳號] [會員公鑰]
```

10. 查看會員可用資源
```
cleos get account [會員帳號]
```

## 撰寫 Hello World 智能合約
1. Hello World
```
#include <eosio/eosio.hpp>

using namespace eosio;

class [[eosio::contract]] helloworld : public contract {
    public:
        using contract::contract;

    [[eosio::action]]
    void hi(name user) {
        print("Hi! ", user, " Hello World!!!");
    }
};
```

2. 編譯智能合約
```
eosio-cpp -abigen -o [智能合約所在資料夾]/helloworld.wasm [智能合約所在資料夾]/helloworld.cpp
```

3. 查看可用公鑰
```
cleos wallet keys
```

4. 新增帳號發布智能合約，EOS當中，一個帳號只能發佈一份智能合約
```
cleos create account eosio [會員帳號] [會員公鑰] -p eosio@active
```

5. 發佈智能合約
```
cleos set contract [智能合約名稱] [智能合約在屬資料夾] -p [會員帳號]@active
```

6. 調用智能合約，調用智能合約與上合約的不能同一人
```
cleos push action [智能合約名稱] [調用的 Action] '[[代入參數]]' -p [會員帳號]@active
```

## 撰寫資料長期儲存智能合約
1. 上傳打卡鐘資料
```
#include <eosio/eosio.hpp>

using namespace eosio;

class [[eosio::contract("punchclock")]] punchclock : public eosio::contract {
	public:
		punchclock(name receiver, name code,  datastream<const char*> ds): contract(receiver, code, ds) {}

		[[eosio::action]] //新增資料
		void add(std::string punchTime, std::string doorNo, std::string cardNo, std::string identify, name user) {
			require_auth(user);
			punchListIndex lists(get_self(), get_first_receiver().value);
			lists.emplace(user, [&](auto& row) {
				row.id = lists.available_primary_key();
				row.creater = user.value;
				row.punchtime = punchTime;
				row.doorno = doorNo;
				row.cardno = cardNo;
				row.identify = identify;
			});
		}

		[[eosio::action]] //刪除資料
		void remove(uint64_t id, name user) {
			require_auth(user);
			punchListIndex lists(get_self(), get_first_receiver().value);
			auto iterator = lists.find(id);
			check(iterator != lists.end(), "Record does not exist");
			lists.erase(iterator);
		}
		
		[[eosio::action]]
		void get(std::string identify, name user) {
		    require_auth(user);
		    punchListIndex lists(get_self(), get_first_receiver().value);
		    auto iterator = lists.begin();
		    while(iterator != lists.end()) {
		        if (identify == iterator->identify) {
		            print(iterator->identify);
		        }
		    }
		}

	private:
		struct [[eosio::table]] punchList { //punchList資料表
			uint64_t id;
			uint64_t creater;
			std::string punchtime;
			std::string doorno;
			std::string cardno;
			std::string identify;
			uint64_t primary_key() const { return id; }
			uint64_t get_secondary_1() const { return creater; }
		};
		typedef eosio::multi_index<"punchclock"_n, punchList,
		    indexed_by<"bycreater"_n, const_mem_fun<punchList, uint64_t, &punchList::get_secondary_1>>
		> punchListIndex;
};
```

2. 調用智能合約
```
cleos push action punchclock add '["2019-06-08 11:28:00", "A01", "ABCDEFG", "00001", "bob"]' -p bob@active
```

3. 查看線上資料庫
```
root@a9e45804cee8:~/contracts# cleos get table [會員帳號] [上鏈的會員帳號] [資料表名]
```

## EOSIO 開發工具
1. 線上編輯器
https://beosin.com/BEOSIN-IDE/index.html#/

2. 測試鏈
https://monitor.jungletestnet.io/

## IPFS管理介面
http://127.0.0.1:5001/webui

PS. 不想連外，可以到後台Settings裡面，將Bootstrap陣列中資料清空
