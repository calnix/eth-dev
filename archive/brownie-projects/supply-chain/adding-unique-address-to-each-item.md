---
description: >-
  https://medium.com/upstate-interactive/creating-a-contract-with-a-smart-contract-bdb67c5c8595
---

# Adding unique address to each item

We want each item that was created to have its own unique address, to which customers can make payment.

I initially thought:

```solidity
    struct Item {
        string id;
        uint price;
        itemState state;
        address _payto;
    }
```

However, this is would not work. How do we define/pre-allocate an address automatically to each item on creation? We do not control address creation and allocation on the EVM.

## Solution

To get a unique address per item -> each item has its own smart contract. Therefore, the create() should result in deployment on a new item AND its smart contract.

We will split the code up, keeping the item smart contract as a separate child contract.&#x20;

{% tabs %}
{% tab title="item.sol" %}
```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.1;
import "./ItemManager.sol";

contract Item {
    uint public priceInWei;
    uint public paidWei;
    uint public index;
    
    //variable of type ItemManager -> contract object.
    ItemManager parentContract;     
    
    // creation requires inputs: price & index.
    constructor(ItemManager _parentContract, uint _priceInWei, uint _index) public {
        priceInWei = _priceInWei;
        index = _index;
        parentContract = _parentContract;
    }
    receive() external payable {
        require(msg.value == priceInWei, "We don't support partial payments");
        require(paidWei == 0, "Item is already paid!");
        paidWei += msg.value;
        (bool success, ) = address(parentContract).call{value:msg.value}(abi.encodeWithSignature("triggerPayment(uint256)", index));
        require(success, "Delivery did not work");
    }

    fallback () external {
    }
}
```
{% endtab %}

{% tab title="ItemManager.sol" %}
```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.1;

contract ItemManager {
   
   enum itemState{
        Created, Paid, Delivered    //created - 0, paid -1,..
    }

    struct Item {
        Item _item;         //_item variable is contract object? by above logic
        string id;
        uint price;
        itemState state;
    }
    
    // to create a dataframe-like structure
    mapping(uint => Item) public item_list;
    uint item_index;
    
    event SupplyChainStep(uint _index, uint _state);

    function createItem(string memory _id, uint _price) public {
        Item item = new Item(this, _price, item_index);   //create new Item contract. this: the current contract, explicitly convertible to address
        item_list[item_index]._item = item;

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
{% endtab %}
{% endtabs %}

### How do these 2 contracts interact?

#### createItem

When createItem() is called `Item item = new Item(this, _price, item_index)`, creates a new instance of the Item contract based on the price provided and index counter.&#x20;

`this` : the current contract, explicitly convertible to address.

`this` is passed into the child contract's constructor (`Item{}`), for the field `ItemManager _parentContract`, where it eventually assigned to variable parentContract&#x20;

```
parentContract = _parentContract; 
// evaluates to:
parentContract = this;
```

This is to store the parent contract association.&#x20;

#### struc Item {}

struc Item now has `Item _item`; which is a contract object. It is assigned value under the createItem(), after the new item contract object has been created.

`item_list[item_index]._item = item;`



### `Thoughts`

This approach allows us to create an item using the parent contract, assign values to both parent and child
