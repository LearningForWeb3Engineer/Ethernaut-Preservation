# Ethernaut-Preservation
Ethernaut Preservation解題思路

<img width="1448" height="1518" alt="image" src="https://github.com/user-attachments/assets/560d083b-3564-4ba1-8823-43e666b57559" />


***這題需熟練delegatecall的用法***

這題的要求是要把owner改成自己，並且有提示用delegatecall的方式，在執行這一題需知道delegatecall是如何改變各個合約的變數的

在Solidity儲存都是用slot來儲存有順序的宣告變數，在原始的合約裡面我們可以發現：

slot0:timeZone1Library
slot1:timeZone2Library
slot2:owner
slot3:storedTime

並且可以發現他的低階呼叫是透過timeZone1Library這個合約去呼叫setTime(uint256 _time)，且在setTime(uint256 _time)裡面可以發現變數storedTime位於該合約的slot0

這意味著當執行delegatecall時會改變合約Preservation合約裡面slot0的變數也就是timeZone1Library

所以我們要做的事就是建立一個攻擊合約先執行第一次setFirstTime讓timeZone1Library先變成自己的攻擊合約，再執行第二次setFirstTime讓自己攻擊合約的owner可以對到原本合約Preservation的owner(slot2)

進而改變owner變成自己，這樣就可以通過了

Tips:

*1.delegatecall會改變的原本呼叫的合約裡面指定的的slot槽的內容

*2.delegatecall的傳入方式是看前byte4，所以在傳入之前需要先用encode進行編碼

關於encode可以分為四種方式：

abi.encode

abi.encodePacked

abi.encodeWithSelector

abi.encodeWithSignature

附上攻擊合約程式碼：

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract Attack{
        address public timeZone1Library;
        address public timeZone2Library;
        address public owner;
    function setTime(uint256 _time) public {
        owner=address(uint160(_time));
    }
