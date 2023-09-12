// SPDX-License-Identifier: Unlicensed

pragma solidity 0.8.19;


//Peer-to-Peer Contract
contract P2PContract {

    //Struct containing variables to store prosumer information: current energy, current balance, 
    //a boolean to check if the user is registered and current tokens.
    struct Prosumer {
        int currentEnergy;
        uint currentBalance;
        bool checkIfRegistered;
        uint currentTokens;
    }

    //A public dynamic array to store prosumer addresses as well as a mapping that maps each address
    //to a Prosumer struct.
    address[] public registeredAddresses;
    mapping (address => Prosumer) public prosumers;

    //A series of events that are emitted whenever energy is sold or purchased, whenever a new prosumer is
    //registered, and whenever a prosumer obtains tokens.  
    event EnergyTradeSold (address seller, int energy);
    event EnergyTradePurchased (address buyer, int energy);
    event RegisteredAddress(address prosumer);
    event TokensReceived(address prosumer, uint tokens);

    //A function to register a new prosumer. First it checks if the prosumer has already been registered. If
    //so, a message is displayed saying that the prosumer is already registered. Otherwise, the new prosumer 
    //is added and stored in the registeredAddresses dynamic array.
    function registerProsumer() internal {
        if (prosumers[msg.sender].checkIfRegistered) {
        revert("Prosumer is already registered");
        } else {
            Prosumer memory newprosumer = Prosumer(0, 0, true, 0);
            prosumers[msg.sender] = newprosumer;
            registeredAddresses.push(msg.sender);
            emit RegisteredAddress(msg.sender);
        }
    }

    //Function used to purchase energy that is called by the energyRequest function in the main contract.
    //It starts by checking that the energy amount entered is greater than zero, and if there are any available
    //sellers. If these requirements are satisfied, It then makes sure that the prosumer's balance is 
    //higher or equal to the amount of energy requested. Once these checks have been performed, it adds the energy 
    //amount to the purchaser's current energy, and removes the balance from their wallet, while simultaneously 
    //removing energy from the seller's current energy and adding the balance to their wallet. It also adds 10 tokens 
    //to the buyer's current amount of tokens (10 tokens per transaction).
    function purchaseEnergy(int energyAmount) internal {
        if (energyAmount <= 0) {
            revert("Energy amount cannot be negative");
        } else {
            address seller = availableProsumers(energyAmount);
            require (seller != address(0), "No sellers available");
            uint totalCost = uint(energyAmount) * 1 ether;
            require(prosumers[msg.sender].currentBalance >= totalCost, "Insufficient funds to make the purchase");
            prosumers[msg.sender].currentEnergy += energyAmount;
            prosumers[msg.sender].currentBalance -= totalCost;
            prosumers[msg.sender].currentTokens += 10;
            prosumers[seller].currentEnergy -= energyAmount;
            prosumers[seller].currentBalance += totalCost;
            prosumers[seller].currentTokens += 10;
            emit EnergyTradePurchased(msg.sender, energyAmount);
            emit TokensReceived(msg.sender, 10);
            }
    }

    //Function used to sell energy that is called by the energyRequest function in the main contract.
    //It starts by checking that the energyAmount for sale is higher than 0, which if satisified, adds that
    //amount to be sold by the seller, as well as adding 10 tokens to the sellers's current amount
    //of tokens (10 tokens per transaction).
    function sellEnergy(int energyAmount) internal {
        if (energyAmount <= 0) {
            revert("Energy amount cannot be negative");
        } else {
            prosumers[msg.sender].currentEnergy += energyAmount;
            emit EnergyTradeSold(msg.sender, energyAmount);
            emit TokensReceived(msg.sender, 10);
        }
    }

    //Function to find any available prosumers for buying or selling energy.
    //The function searches through the registeredAddresses array of registered prosumers, based on the type of
    //request that has been sent out (purchase/sell) and the required amount of energy. If it finds a prosumer that
    //matches the requests parameters, it then returns that prosumers address. If it does not find a prosumer, it 
    //returns a null address.
    function availableProsumers(int requiredEnergy) public view returns(address) {
        for (uint i = 0; i < registeredAddresses.length; i++) {
            address prosumerAddress = registeredAddresses[i];
            if (prosumers[prosumerAddress].checkIfRegistered && prosumers[prosumerAddress].currentEnergy > requiredEnergy) {
                return prosumerAddress;
            } 
        }
        return address(0);
    }

    //A function used to purchase energy with tokens that the user has earned. Unlike the purchaseEnergy and sellEnergy
    //functions, this is not done through the energyRequest function, but rather the prosumer directly inputs the number of
    //energy they wish to purchase. First the function checks if there are any sellers available, it then makes sure that
    //the prosumer has at least 100 tokens in order to make a purchase. (100 tokens per ether). It then adds the purchased
    //amount to the prosumers energy, and deducted tokens from the current tokens.
    function purchaseEnergyWithTokens(int energyAmount) public {
        address seller = availableProsumers(energyAmount);
            if (seller == address(0)) {
                revert("No sellers available");
            } else {
            require(prosumers[msg.sender].currentTokens >= 100, "Prosumer currently has insufficient tokens");
            require(prosumers[msg.sender].currentEnergy + energyAmount >= 0, "");
            prosumers[msg.sender].currentEnergy += energyAmount;
            prosumers[msg.sender].currentTokens -= 100;
            emit EnergyTradePurchased(msg.sender, energyAmount);
            emit TokensReceived(msg.sender, 100);
            }
        }
    }


//Main Contract
contract MainContract is P2PContract {

    //Events to monitor the balance deposited, the balance withdrawn and an invalid amount of energy 
    //entered.
    event BalanceDeposited(uint balance);
    event BalanceWithdrawn(address prosumer, uint balance);
    event InvalidEnergyAmount(address prosumer, int energyAmount);

    //Modifier to only allow function utilisation to registered prosumers 
    modifier isRegistered () {
        require(prosumers[msg.sender].checkIfRegistered, "Prosumer is not yet registered");
        _;
    }

    //Modifier to only allow function utilisation to prosumers that are not yet registered.
    modifier isNotRegistered () {
        require(!prosumers[msg.sender].checkIfRegistered, "Prosumer is already registered");
        _;
    }

    //Modifier to check if prosumer has sufficient funds, prior to purchasing energy.
    //If the user inputs a value of less than zero (a purchase request), then it is required
    //that their balance is higher than or equal two the requested amount of energy.
    modifier hasSufficientFundsOfEnergy(int energyAmount) {
        if (energyAmount < 0) {
            if (prosumers[msg.sender].currentBalance < uint(-energyAmount) * 1 ether) {
                revert("Not enough funds to purchase energy");
            }
        } 
        _;
    }

    //Function that calls the register prosumer function in the P2P Contract and acts 
    //as a button to add a new prosumer in the main contract.
    function addProsumer() public isNotRegistered() {
        registerProsumer();
    }

    //Function to deposit ethers into a registered prosumers wallet in the contract.
    function depositEthers() public isRegistered() payable {
        prosumers[msg.sender].currentBalance += msg.value;
        emit BalanceDeposited(msg.value);
    }

    //Function to withdraw ethers from a registered prosumers wallet in the contract.
    function withdrawBalance() public isRegistered() payable {
        uint amount = prosumers[msg.sender].currentBalance;
        require(amount > 0, "Prosumer does not have enough balance to withdraw");
        prosumers[msg.sender].currentBalance = 0;
        (bool success,) = msg.sender.call{value: amount}("");
        require(success, "Withdrawal failure");
        emit BalanceWithdrawn(msg.sender, amount);
    }

    //Function to send a trading request to either buy or sell. 
    //First it makes sure that the entered amount is not equal to zero. If the user enters a positive number, 
    //it calls the sellEnergy function in the P2P Contract, which puts up an available amount of energy to sell. 
    //If the user enters a negative number, it calls the purchaseEnergy function in the P2P Contract, which buys 
    //the entered amount off of the first available prosumer. Otherwise, if the amount entered is equal to zero then
    //the InvalidEnergyAmount event is emitted.
    function sendRequest(int energyAmount) public isRegistered() hasSufficientFundsOfEnergy(energyAmount) {
        require(energyAmount != 0, "You cannot buy or sell zero energy");
        if (energyAmount < 0) {
            purchaseEnergy(-energyAmount);
        } else if (energyAmount > 0) {
            sellEnergy(energyAmount);
        } else {
            emit InvalidEnergyAmount(msg.sender, energyAmount);
        }
    }

    //Function to check the current balance that the prosumer has.
    function checkProsumerBalance() public view isRegistered() returns (uint) {
        return prosumers[msg.sender].currentBalance;
    }

    //Function to check the current energy that the prosumer has.
    function checkProsumerEnergy() public view isRegistered() returns (int) {
        return prosumers[msg.sender].currentEnergy;
    }

    //Function to check the current number of tokens that the prosumer has.
    function checkProsumerTokens() public view isRegistered() returns (uint) {
        return prosumers[msg.sender].currentTokens;
    }

}
