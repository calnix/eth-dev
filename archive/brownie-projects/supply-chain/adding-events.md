# Adding Events

```
// add event that we can use for each function:
        event SupplyChainStep(uint _index, uint _state);
```

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
    
    event SupplyChainStep(uint _index, uint _state);

    function createItem(string memory _id, uint _price) public {
        item_list[item_index].id = _id;
        item_list[item_index].price = _price;
        item_list[item_index].state = itemState.Created;

        emit SupplyChainStep(item_index, uint(item_list[item_index].state));
        item_index++;

    }

    function triggerPayment(uint _index) public payable {
        require(item_list[_index].price == msg.value, "please pay exact full amount");
        require(item_list[_index].state == itemState.Created,"Item is not available");
        item_list[_index].state = itemState.Paid;   //update state to paid
        // emit payment event
        emit SupplyChainStep(item_index, uint(item_list[item_index].state));
        
        
    }

    function triggerDeliver(uint _index) public {
        require(item_list[_index].state == itemState.Paid,"Item is not for delivery");
        item_list[_index].state == itemState.Delivered;

        // emit delivery
        emit SupplyChainStep(item_index, uint(item_list[item_index].state));
    }
}
```
