# King-Of-The-Hill

**Smartcontract** : https://goerli.etherscan.io/tx/0x96ede89e6bd3123e84d84df17459f7f03f5fba5bb8dde02ff629d8675d1ee714  
**testnet** : goerli



## The game ##

```js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol";

/// @title A game based on the KingOfTheHill's game model
/// @author Guillaume Bézie with major instructions and smartcontract architecture from developer Benmissi.A (https://github.com/Benmissi-A/king-of-the-hill/blob/main/KingOfTheHill.sol)
/// @notice This smartcontract has been developped with testnet and for only simulation purpose.

contract KingOfTheHill{
    using Address for address payable;
    
    address private _owner;             ///@dev owner of the smartcontract
    address private _potOwner;          ///@dev first player
    uint256 private _seed;              ///@dev value of the seed
    uint256 private _blockLimit;        ///@dev define the number of blocks by stage
    uint256 private _startingBlock;     ///@dev block initialisation
    bool private _gameStart;            ///@dev boolean help for intialisation of each game/stage
   
    
    constructor(address owner_ , uint256 blockLimit_) payable {                     ///@owner intialisation of the game with a seed value and a block number
        require(msg.value>0 , "Add a seed before the game's initialisation");       ///@owner revert information guarantee a good initialisation
        _owner = owner_;
        _seed = msg.value;
        _blockLimit = blockLimit_;
    }
    
    event Enthroned(address sender);                                ///@notice currents informations during the game
    event Payed(address indexed recipient, uint256 amount);
    event Refunded(address indexed recipient, uint256 rest);
    
    
    function offer() external payable {                                                                     ///@notice each player interract and play the game
        require(msg.value >= 2 * _seed , "KingOfTheHill: you need to put twice the value of the seed");     ///@notice revert information
        
        if(_gameStart = false){
            _setStartingBlock();
            _gameStart = true;
        }
        
        ///@dev if the player put a bigger value than the game's rule, he can be refunded.
        if(msg.value> 2*_seed){                                     
            uint256 rest = 2*_seed;
            payable(msg.sender).sendValue(msg.value - rest);
            emit Refunded(msg.sender, msg.value - rest);
        }
        _resolveTurn();
        
        _potOwner = msg.sender;
        emit Enthroned(msg.sender);
        _seed = address(this).balance;
    }
    
    
    function _withdrawSeed() private {                              ///@dev function define the winner payment and owner compensation
         _seed = (address(this).balance * 10)/100;                  
         uint256 part = address(this).balance;
         payable(_potOwner).sendValue((part * 80)/100);             ///@dev gain of the winner
         payable(_owner).sendValue((part * 10)/100);                ///@dev owner compensation
         emit Payed(_potOwner, (part * 80)/100);
    }
    
    
        function _setStartingBlock() private {                      /// define a new game
        _startingBlock = block.number;
    }
    
    function _resolveTurn() private {                                       ///@dev control of the stage with time restrictions (block number)
        if(block.number - _startingBlock >= _blockLimit){
            if(_potOwner !=  0x0000000000000000000000000000000000000000 ){
                _withdrawSeed();
            }
            _setStartingBlock();                                    ///@dev game intialisation
        }
    }
    
    
    
    function viewPassedtBlocksNumber()public view returns (uint256) {
        return block.number - _startingBlock;
    }
    
    function getPotOwner() public view returns (address) {
        return _potOwner;
    }
    
    function getSeed() public view returns (uint256) {
        return _seed;
    }
    
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
    
    function  viewBlockLimit()public view returns(uint256) {
        return _blockLimit;
    }
    
    function viewStartingBlock()public view returns (uint256) {
        return _startingBlock;
    }
    
    function viewCurrentBlock()public view returns (uint256) {
        return block.number;
    }
    
    function withdrawAll() public {
        require(msg.sender == _owner , "Only Owner");
         _seed = 0;
         payable(_owner).sendValue(address(this).balance);
    }
    
}