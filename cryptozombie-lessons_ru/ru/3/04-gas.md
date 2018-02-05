---
title: Газ
actions: ['Проверить', 'Подсказать']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiefactory.sol": |
        pragma solidity ^0.4.19;

        import "./ownable.sol";

        contract ZombieFactory is Ownable {

            event NewZombie(uint zombieId, string name, uint dna);

            uint dnaDigits = 16;
            uint dnaModulus = 10 ** dnaDigits;

            struct Zombie {
                string name;
                uint dna;
                // Здесь добавь новые данные
            }

            Zombie[] public zombies;

            mapping (uint => address) public zombieToOwner;
            mapping (address => uint) ownerZombieCount;

            function _createZombie(string _name, uint _dna) internal {
                uint id = zombies.push(Zombie(_name, _dna)) - 1;
                zombieToOwner[id] = msg.sender;
                ownerZombieCount[msg.sender]++;
                NewZombie(id, _name, _dna);
            }

            function _generateRandomDna(string _str) private view returns (uint) {
                uint rand = uint(keccak256(_str));
                return rand % dnaModulus;
            }

            function createRandomZombie(string _name) public {
                require(ownerZombieCount[msg.sender] == 0);
                uint randDna = _generateRandomDna(_name);
                randDna = randDna - randDna % 100;
                _createZombie(_name, randDna);
            }

        }
      "zombiefeeding.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefactory.sol";

        contract KittyInterface {
          function getKitty(uint256 _id) external view returns (
            bool isGestating,
            bool isReady,
            uint256 cooldownIndex,
            uint256 nextActionAt,
            uint256 siringWithId,
            uint256 birthTime,
            uint256 matronId,
            uint256 sireId,
            uint256 generation,
            uint256 genes
          );
        }

        contract ZombieFeeding is ZombieFactory {

          KittyInterface kittyContract;

          function setKittyContractAddress(address _address) external onlyOwner {
            kittyContract = KittyInterface(_address);
          }

          function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            if (keccak256(_species) == keccak256("kitty")) {
              newDna = newDna - newDna % 100 + 99;
            }
            _createZombie("NoName", newDna);
          }

          function feedOnKitty(uint _zombieId, uint _kittyId) public {
            uint kittyDna;
            (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
            feedAndMultiply(_zombieId, kittyDna, "kitty");
          }

        }
      "ownable.sol": |
        /**
         * @title Ownable
         * @dev The Ownable contract has an owner address, and provides basic authorization control
         * functions, this simplifies the implementation of "user permissions".
         */
        contract Ownable {
          address public owner;

          event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

          /**
           * @dev The Ownable constructor sets the original `owner` of the contract to the sender
           * account.
           */
          function Ownable() public {
            owner = msg.sender;
          }

          /**
           * @dev Throws if called by any account other than the owner.
           */
          modifier onlyOwner() {
            require(msg.sender == owner);
            _;
          }

          /**
           * @dev Allows the current owner to transfer control of the contract to a newOwner.
           * @param newOwner The address to transfer ownership to.
           */
          function transferOwnership(address newOwner) public onlyOwner {
            require(newOwner != address(0));
            OwnershipTransferred(owner, newOwner);
            owner = newOwner;
          }

        }
    answer: >
      pragma solidity ^0.4.19;

      import "./ownable.sol";

      contract ZombieFactory is Ownable {

          event NewZombie(uint zombieId, string name, uint dna);

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;

          struct Zombie {
              string name;
              uint dna;
              uint32 level;
              uint32 readyTime;
          }

          Zombie[] public zombies;

          mapping (uint => address) public zombieToOwner;
          mapping (address => uint) ownerZombieCount;

          function _createZombie(string _name, uint _dna) internal {
              uint id = zombies.push(Zombie(_name, _dna)) - 1;
              zombieToOwner[id] = msg.sender;
              ownerZombieCount[msg.sender]++;
              NewZombie(id, _name, _dna);
          }

          function _generateRandomDna(string _str) private view returns (uint) {
              uint rand = uint(keccak256(_str));
              return rand % dnaModulus;
          }

          function createRandomZombie(string _name) public {
              require(ownerZombieCount[msg.sender] == 0);
              uint randDna = _generateRandomDna(_name);
              randDna = randDna - randDna % 100;
              _createZombie(_name, randDna);
          }

      }
---

Круто! Теперь мы знаем, как обновлять ключевые части DApp, при этом заставляя других пользователей держаться подальше от наших контрактов, чтобы не испортили. Now we know how to update key portions of the DApp while preventing other users from messing with our contracts.

Давай посмотрим на еще одно серьезное отличие Solidity от других языков программирования:

## Газ — это топливо для DApps на Ethereum

В Solidity пользователи должны заплатить за каждый раз, когда они вызывают функцию вашего DApp, с помощью валюты под названием ** _газ_ **. Газ покупают вместе с эфиром, валютой Ethereum, таким образом пользователи платят ETH, чтобы выполнить функцию приложения DApp.In Solidity, your users have to pay every time they execute a function on your DApp using a currency called **_gas_**. Users buy gas with Ether (the currency on Ethereum), so your users have to spend ETH in order to execute functions on your DApp. 

Количество газа для выполнения функции зависит от сложности логики функции. У любой операции есть **_стоимость газа_**, она основана на количестве вычислительных ресурсов, необходимых для выполнения  операции (например, запись в хранилище намного дороже, чем добавление двух целых чисел). Общая **_стоимость газа_** вашей функции - сумма затрат газа на все операции.

How much gas is required to execute a function depends on how complex that function's logic is. Each individual operation has a **_gas cost_** based roughly on how much computing resources will be required to perform that operation (e.g. writing to storage is much more expensive than adding two integers). The total **_gas cost_** of your function is the sum of the gas costs of all its individual operations.

Поскольку запуск функций стоит реальных денег пользователям, оптимизация кода гораздо важнее в Ethereum, чем в других языках программирования. Если код написан небрежно, а пользователям придется платить за выполнение функций, в пересчете на тысячи пользователей это может означать миллионы долларов ненужных комиссий.

Because running functions costs real money for your users, code optimization is much more important in Ethereum than in other programming languages. If your code is sloppy, your users are going to have to pay a premium to execute your functions — and this could add up to millions of dollars in unnecessary fees across thousands of users.

## Зачем нужен газ?

Ethereum похож на большой, медленный, но крайне безопасный компьютер. Когда ты выполняешь функцию, каждая нода в сети должна запустить ту же самую функцию проверки результата на выходе. Это то, что делает Ethereum децентрализованным, а данные в нем неизменяемыми а не подверженными цензуре.

Ethereum is like a big, slow, but extremely secure computer. When you execute a function, every single node on the network needs to run that same function to verify its output — thousands of nodes verifying every function execution is what makes Ethereum decentralized, and its data immutable and censorship-resistant.

Создатели Ethereum хотели быть уверенными, что никто не сможет заспамить сеть, запустив бесконечный цикл, или сожрать все сетевые ресурсы интенсивными вычислениями. Поэтому они сделали транзакции платными — пользователи должны платить за использование вычислительных мощностей и за хранение.

The creators of Ethereum wanted to make sure someone couldn't clog up the network with an infinite loop, or hog all the network resources with really intensive computations. So they made it so transactions aren't free, and users have to pay for computation time as well as storage.

> Обрати внимание. Для сайдчейнов, как например для того, который авторы КриптоЗомби используют в Loom Network, это правило не обязательно. Нет смысла запускать игру вроде World of Warcraft в главной сети Ethereum - стоимость газа будет реальным барьером. Но зато такая игра может работать на сайдчейне с другим алгоритмом консенсус. В следующих уроках мы вернемся к тому, какие DApps развертывать на сайдчейне, а какие в главной сети Ethereum.

> Note: This isn't necessarily true for sidechains, like the ones the CryptoZombies authors are building at Loom Network. It probably won't ever make sense to run a game like World of Warcraft directly on the Ethereum mainnet — the gas costs would be prohibitively expensive. But it could run on a sidechain with a different consensus algorithm. We'll talk more about what types of DApps you would want to deploy on sidechains vs the Ethereum mainnet in a future lesson.

## Как упаковать структуру, чтобы сэкономить газ

Мы упоминали в первом уроке, что есть разные типы `uint`: `uint8`,` uint16`, `uint32` и так далее. 

Обычно использование этих подтипов нецелесообразно, поскольку Solidity резервирует 256 бит в хранилище независимо от размера `uint`. Например, использование `uint8` вместо `uint` (`uint256`) не экономит газ.

Но есть и исключение внутри структур.

Если внутри структуры несколько `uint`, использование по возможности `uint` меньшего размера позволит Solidity объединить переменные вместе и уменьшить объем хранилища. Например:

```
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` будет стоить меньше, чем `normal` из-за упаковки структуры
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```

Так что внутри структур можешь использовать наименьшие целочисленные подтипы, которые позволяют запустить код.

Еще можно объединить в кластеры идентичные типы данных, то есть поставить их в структуре рядом друг с другом. Так Solidity оптимизирует требующееся пространство в хранилище. К примеру, структура поля `uint c; uint32 a; uint32 b;` будет стоить меньше газа, чем структура с полями `uint32 a; uint c; uint32 b;`, потому что поля `uint32` группируются вместе.

## Проверь себя

В этом уроке мы собираемся добавить зомби 2 новые фишки: `level` (уровень) и `readyTime` (время готовности) — послдняя будет использоваться для установки времени восстановления, чтобы ограничить частоту питания зомби. 

Вернемся назад к `zombiefactory.sol`.

1. Добавь еще два свойства к структуре `Zombie`: `level` (`uint32`) и `readyTime` (тоже `uint32`). Мы хотим объединить эти типы данных, давай поместим их в конец структуры.

32 бита достаточно для хранения уровня и времени восстановления зомби. Мы сэкономили немного газа, упаковав данные поплотнее, чем обычный 256-битный `uint`.
