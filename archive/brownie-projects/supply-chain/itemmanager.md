# ItemManager()



```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.1;

contract ItemManager {
   
   enum itemState{
        Created, Paid, Delivered    //created - 0, paid -1,..
    }

    struct Item {
        string id;
        uint price;
        itemState state;
    }
    
    // to create a dataframe structure
    mapping(uint => Item) public item_list;
    uint item_index;
    

    function createItem(string memory _id, uint _price) public {
        item_list[item_index].id = _id;
        item_list[item_index].price = _price;
        item_list[item_index].state = itemState.Created;
        item_index++;
    }

    function triggerPayment(uint _index) public payable {
        require(item_list[_index].price == msg.value, "please pay exact full amount");
        require(item_list[_index].state == itemState.Created,"Item is not available");
        item_list[_index].state = itemState.Paid;   //update state to paid        
    }

    function triggerDeliver(uint _index) public {
        require(item_list[_index].state == itemState.Paid,"Item is not for delivery");
        item_list[_index].state == itemState.Delivered;
    }
}

```

### struct & enum

First we create the item object with the struct item{}, need to reflect the following properties of id, price, address payable and the state.

State here refers to which stage in the supply-chain the item is currently at: Created, Paid, Delivered.

To achieve this we use `enum itemState{Created, Paid, Delivered}`.

### mapping

We would need a dataframe like structure to track out inventory of items, therefore a mapping from a `uint` to struct `Item`, where the uint is the index. We achieve this index capability with the creation of item\_index -> we will increment this later and feed into the mapping. count starts at 0.

### function createItem()

When we create an item, we are looking to update the mapping `item_list` -> the mapping is initialized with default values of all 0. So we are updating each default struct in the mapping with proper values, sequentially starting from 0. Hence:

```solidity
item_list[item_index].id = _id; //and so forth
/* resolves to:
Item.id = _id
*/
```

We do not bother with individually instantiating each item with a unique name and then subsequently populating the item\_list. Instead we simply update the mapping with the relevant details.&#x20;

Why? This saves gas and overheads and we do not have the need to outright declare each item as an object for further manipulation. Simply need to track state and existence.&#x20;
