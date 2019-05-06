## Reentrancy_MAVS_1:   

### Deployment Address:



#### Transaction Counts:



#### Exploitation Code: [here](https://github.com/mavspublic/Exploit_Code/tree/master/Reentrancy/Reentrancy-AVS-01)

#### Reports by existing tools:

| Slither          | Oyente | Smartcheck | MAVS |
| ---------------- | ------ | ---------- | ---- |
| $$ \checkmark $$ | **X**  |            |      |

## **MAVS_2:**

### **Vulnerability_Instance_1:**

​    //out7852.sol

   function buyFirstTokens(

​        IMultiToken _mtkn,

​        bytes _callDatas,

​        uint[] _starts // including 0 and LENGTH values

​                                              )

​        Public  payable

​    {

​        change(_callDatas, _starts);

 

​        uint tokensCount = _mtkn.tokensCount();

​        uint256[] memory amounts = new uint256[](tokensCount);

​        for (uint i = 0; i < tokensCount; i++) {

​            ERC20 token = _mtkn.tokens(i);

​            amounts[i] = token.balanceOf(this);

​            if (token.allowance(this, _mtkn) == 0) {

​                token.asmApprove(_mtkn, uint256(-1));

​            }

​        }

 

​        _mtkn.bundleFirstTokens(msg.sender, msg.value.mul(1000), amounts);

​        if (address(this).balance > 0) {

​            msg.sender.transfer(address(this).balance);

​        }

​        for (i = _mtkn.tokensCount(); i > 0; i--) {

​            token = _mtkn.tokens(i - 1);

​            token.asmTransfer(msg.sender, token.balanceOf(this));

​        }

​    }

 

 

## **MAVS_3:**

### **Vulnerability_Instance_1:      //out5455.sol**

 function sendEthProportion(address target, bytes data, uint256 mul, uint256 div) external {

​        uint256 value = address(this).balance.mul(mul).div(div);

​        // solium-disable-next-line security/no-call-value

​        require(target.call.value(value)(data));            ///////////////////////////////////////

​    }

### **Vulnerability_Instance_2:             //7852.sol**

function sendEthProportion(address _target, bytes _data, uint256 _mul, uint256 _div) external {

​        uint256 value = address(this).balance.mul(_mul).div(_div);

​        // solium-disable-next-line security/no-call-value

​        require(_target.call.value(value)(_data));          ////////////////////////////////////////

}

 

 

## **MAVS_4:**

### **Vulnerability_Instance_1:            //out5369.sol(****有待提高****)**

function invest() notOnPause public payable {

​        admin.transfer(msg.value * 5 / 100);

​        marketing.transfer(msg.value / 10);

  if (x.d(msg.sender) > 0) {

​            withdraw();

​        }

  x.updateInfo(msg.sender, msg.value);       

​        if (msg.data.length == 20) {

​            toReferrer(msg.value);

​        }

​        emit LogInvestment(msg.sender, msg.value);

​    }

### **V****ulnerability_Instance_2:             //out4793.sol(****有待提高****)**

function invest() notOnPause public payable {

 

​        admin.transfer(msg.value * 8 / 100);

​        marketing.transfer(msg.value * 5 / 100);

 

​        if (x.d(msg.sender) > 0) {

​            withdraw();

​        }

 

​        x.updateInfo(msg.sender, msg.value);    //  这个x不好被操控

 

​        if (msg.data.length == 20) {

​            toReferrer(msg.value);

​        }

 

​        emit LogInvestment(msg.sender, msg.value);

​    }

##  MAVS_5:**

### **Vulnerability_Instance_1:      //out6227.sol**

```javascript
 buy(bytes8 referralCode) internal {

​        require(msg.value>=minEthValue);

​        require(now < saleEnd4); // main sale postponed

 

​        // distribution for referral

​        uint256 remainEth = msg.value;

​        if (referral[referralCode] != msg.sender && renownedPlayers[referral[referralCode]].isRenowned)

​        {

​            uint256 referEth = msg.value.mul(10).div(100);

​            referral[referralCode].transfer(referEth);

​            remainEth = remainEth.sub(referEth);

​        }

 

​        if (!renownedPlayers[msg.sender].isRenowned)

​        {

​            generateRenown();

​        }

​        

​        uint256 amount = manager.getYumerium(msg.value, msg.sender);        //////////////////////////

​        uint256 total = totalSaled.add(amount);

​        owner.transfer(remainEth);

​        

​        require(total<=maxSale);

​        

​        totalSaled = total;

​        

​        emit Contribution(msg.sender, amount);

​    }


```

vulnerable code:

` emit Contriution` this code has vulnerability .

**  can leverage this vul to attack this contract

###  Vulnerability_Instance_2:**

​    function buy() internal {

​        require(msg.value>=minEthValue);

​        require(now < saleEnd4); // main sale postponed

​        

​        uint256 amount = manager.getYumerium.value(msg.value)(msg.sender);   

​        uint256 total = totalSaled.add(amount);

​        

​        require(total<=maxSale);

​        

​        totalSaled = total;

​        if (currentDay > 0) {

​            eachDaySold[currentDay] = eachDaySold[currentDay].add(msg.value);

​            uint256 tickets = msg.value.div(10 ** 17);

​            if (ticketsEarned[currentDay][msg.sender] == 0) {

​                eventSaleParticipants[currentDay].push(msg.sender);

​            }

​            ticketsEarned[currentDay][msg.sender] = ticketsEarned[currentDay][msg.sender].add(tickets);

​            totalTickets[currentDay] = totalTickets[currentDay].add(tickets);

​            if (now >= saleEnd3)

​            {

​                currentDay = 0;

​            }

​            else if (now >= saleEnd2)

​            {

​                currentDay = 3;

​            }

​            else if (now >= saleEnd1)

​            {

​                currentDay = 2;

​            }

​        }

​        

​        emit Contribution(msg.sender, amount);

​    }

##  MAVS_6:

### **Vulnerability_Instance_1:   out4115.sol**

 function convertSafe(

​        TokenConverter converter,

​        Token fromToken,

​        Token toToken,

​        uint256 amount

​    ) internal returns (uint256 bought) {

​        if (fromToken != ETH_ADDRESS) require(fromToken.approve(converter, amount));

​        uint256 prevBalance = toToken != ETH_ADDRESS ? toToken.balanceOf(this) : address(this).balance;

​        uint256 sendEth = fromToken == ETH_ADDRESS ? amount : 0;

​        uint256 boughtAmount = converter.convert.value(sendEth)(fromToken, toToken, amount, 1);//////////////////////////

​        require(

​            boughtAmount == (toToken != ETH_ADDRESS ? toToken.balanceOf(this) : address(this).balance) - prevBalance,

​            "Bought amound does does not match"

​        );

​        if (fromToken != ETH_ADDRESS) require(fromToken.approve(converter, 0));

​        return boughtAmount;

​    }

 

## **MAVS_7:**

### **Vulnerability_Instance_1:    out5508.sol**

​        function lend(address to, ERC20 token, uint256 amount, address target, bytes data) public payable {

​        uint256 prevBalance = token.balanceOf(this);

​        token.asmTransfer(to, amount);

​        _inLendingMode += 1;

​        require(caller().makeCall.value(msg.value)(target, data), "lend: arbitrary call failed");

​        _inLendingMode -= 1;

​        require(token.balanceOf(this) >= prevBalance, "lend: lended token must be refilled"); //////////////////////

}

 

 

## **MAVS_8:**

### **Vulnerability_Instance_1:    out7590.sol**

​     function withdraw (address account, address tokenAddr, uint256 index_from, uint256 index_to) external returns (bool) {

​        require(account != address(0x0));

 

​        uint256 release_amount = 0;

​        for (uint256 i = index_from; i < lockedBalances[account][tokenAddr].length && i < index_to + 1; i++) {

​            if (lockedBalances[account][tokenAddr][i].balance > 0 &&

​                lockedBalances[account][tokenAddr][i].releaseTime <= block.timestamp) {

 

​                release_amount = release_amount.add(lockedBalances[account][tokenAddr][i].balance);

​                lockedBalances[account][tokenAddr][i].balance = 0;

​            }

​        }

 

​        require(release_amount > 0);

 

​        if (tokenAddr == 0x0) {

​            if (!account.send(release_amount)) {

​                revert();

​            }

​            emit Withdraw(account, tokenAddr, release_amount);

​            return true;

​        } else {

​            if (!ERC20Interface(tokenAddr).transfer(account, release_amount)) { //////////////////////

​                revert();

​            }

​            emit Withdraw(account, tokenAddr, release_amount);

​            return true;

​        }

​    }

 

## **MAVS_9:**

### **Vulnerability_Instance_1:    10764.sol**

function buyInternal(

​        ERC20 token,

​        address _exchange,

​        uint256 _value,

​        bytes _data

​    ) 

​        internal

​    {

​        require(

​            // 0xa9059cbb - transfer(address,uint256)

​            !(_data[0] == 0xa9 && _data[1] == 0x05 && _data[2] == 0x9c && _data[3] == 0xbb) &&

​            // 0x095ea7b3 - approve(address,uint256)

​            !(_data[0] == 0x09 && _data[1] == 0x5e && _data[2] == 0xa7 && _data[3] == 0xb3) &&

​            // 0x23b872dd - transferFrom(address,address,uint256)

​            !(_data[0] == 0x23 && _data[1] == 0xb8 && _data[2] == 0x72 && _data[3] == 0xdd),

​            "buyInternal: Do not try to call transfer, approve or transferFrom"

​        );

​        uint256 tokenBalance = token.balanceOf(this);

​        require(_exchange.call.value(_value)(_data));           //////////////////////////////////////////////////////

​        balances[msg.sender] = balances[msg.sender].sub(_value);

​        tokenBalances[msg.sender][token] = tokenBalances[msg.sender][token]

​            .add(token.balanceOf(this).sub(tokenBalance));

​    }

 

 

## **MAVS_10:**

### **Vulnerability_Instance_1:      out11664.sol**

function doSupplierTrade(

​        ERC20 src,

​        uint amount,

​        ERC20 dest,

​        address destAddress,

​        uint expectedDestAmount,

​        SupplierInterface supplier,

​        uint conversionRate,

​        bool validate

​    )

​        internal

​        returns(bool)

​    {

​        uint callValue = 0;

​        

​        if (src == ETH_TOKEN_ADDRESS) {

​            callValue = amount;

​        } else {

​            // take src tokens to this contract

​            require(src.transferFrom(msg.sender, this, amount));

​        }

 

​        // supplier sends tokens/eth to network. network sends it to destination

 

​        require(supplier.trade.value(callValue)(src, amount, dest, this, conversionRate, validate));

​        emit SupplierTrade(callValue, src, amount, dest, this, conversionRate, validate);

 

​        if (dest == ETH_TOKEN_ADDRESS) {

​            destAddress.transfer(expectedDestAmount);

​        } else {

​            require(dest.transfer(destAddress, expectedDestAmount));        ///////////////////////////////////////////////////////

​        }

 

​        return true;

​    }

 

 

## **MAVS_11:**

### **Vulnerability_Instance_1:      out12116.sol**

​        function buy(uint256 UniqueID) external payable {

​        address _to = msg.sender;

​        require(TokenIdtosetprice[UniqueID] == msg.value);

​        TokenIdtoprice[UniqueID] = msg.value;

​        uint _blooming = msg.value.div(20);

​        uint _infrastructure = msg.value.div(20);

​        uint _combined = _blooming.add(_infrastructure);

​        uint _amount_for_seller = msg.value.sub(_combined);

​        require(tokenOwner[UniqueID].call.gas(99999).value(_amount_for_seller)());          //////////////////////////////////////

​        this.transferFrom(tokenOwner[UniqueID], _to, UniqueID);

​        if(!INFRASTRUCTURE_POOL_ADDRESS.call.gas(99999).value(_infrastructure)()){

​            revert("transfer to infrastructurePool failed");

​                     }

}

 

 

## **MAVS_12:**

### **Vulnerability_Instance_1:      out1238.sol**

function purchaseFor(address pack, address[] memory users, uint16 packCount, address referrer) public payable {

​        uint price = PackInterface(pack).calculatePrice(PackInterface(pack).basePrice(), packCount);

​          for (uint i = 0; i < users.length; i++) {

​            PackInterface(pack).purchaseFor.value(price)(users[i], packCount, referrer);        /////////////////////////////////////////////

​        }

​    }

 

## MAVS_12:**

### **Vulnerability_Instance_1:      out13135.sol**

 function claimTokens(address _token, address _to) public onlyDonationAddress {  //modifier

​        require(_to != address(0), "Wallet format error");

​        if (_token == address(0)) {

​            _to.transfer(address(this).balance);

​            return;

​        }

 

​        ERC20Basic token = ERC20Basic(_token);

​        uint256 balance = token.balanceOf(this);

​        require(token.transfer(_to, balance), "Token transfer unsuccessful");

​    }

 

## **MAVS_13:**

### **Vulnerability_Instance_1:      out13986.sol**

function Initiate(address _swapadd, uint _amount) payable public{

​        require(msg.value == _amount.mul(2));

​        swap = TokenToTokenSwap_Interface(_swapadd);

​        address token_address = factory.token();

​        baseToken = Wrapped_Ether(token_address);

​        baseToken.createToken.value(_amount.mul(2))();             

​        baseToken.transfer(_swapadd,_amount.mul(2));

​        swap.createSwap(_amount, msg.sender);

​    }

 

 

## **MAVS_14:**

### **Vulnerability_Instance_1:      out151.sol**

 function processPayment(address _address) public {

​        Participant participant = Participant(participants[_address]);

 

​        bool done = participant.processPayment.value(participant.daily())();        

​        

​        if (done) {

​            participants[_address] = address(0);

​            emit ParticipantRemoved(_address);

​        }

​    }

 

 

 

## **MAVS_15:**

### **Vulnerability_Instance_1:      out8253.sol**

function withdrawEther() public {

​        if (roundFailedToStart == true) {

​            require(msg.sender.send(deals[msg.sender].sumEther));

​        }

​        if (msg.sender == operator) {

​            require(projectWallet.send(ethForMilestone+postDisputeEth));

​            ethForMilestone = 0;

​            postDisputeEth = 0;

​        }

​        if (msg.sender == juryOnlineWallet) {

​            require(juryOnlineWallet.send(etherAllowance));

​            require(jotter.call.value(jotAllowance)(abi.encodeWithSignature("swapMe()")));   

​            etherAllowance = 0;

​            jotAllowance = 0;

​        }

​        if (deals[msg.sender].verdictForInvestor == true) {

​            require(msg.sender.send(deals[msg.sender].sumEther - deals[msg.sender].etherUsed));

​        }

​    }

 

 

## **MAVS_15:**

### **Vulnerability_Instance_1:      out10224.sol**

​      function buybackTypeOne() public {

​        uint256 allowanceToken = token.allowance(msg.sender,this);

​        require(allowanceToken != uint256(0));

​        require(isInvestTypeOne(msg.sender));

​        require(isBuyBackOne());

​        require(balancesICOToken[msg.sender] >= allowanceToken);

​        

​        uint256 forTransfer = allowanceToken.mul(buyPrice).div(1e18).mul(3); //calculation Eth 100% in 3 year 

​        require(totalFundsAvailable >= forTransfer);

​        msg.sender.transfer(forTransfer);

​        totalFundsAvailable = totalFundsAvailable.sub(forTransfer);

​        

​        balancesICOToken[msg.sender] = balancesICOToken[msg.sender].sub(allowanceToken);

​        token.transferFrom(msg.sender, this, allowanceToken;

   }

 

 

## **MAVS_16:**

### **Vulnerability_Instance_1:    out940.sol**

function internalContribution(address _contributor, uint256 _wei) internal {

​        updateState();

​        require(currentState == State.InCrowdsale);

 

​        ICUStrategy pricing = ICUStrategy(pricingStrategy);

​        uint256 usdAmount = pricing.getUSDAmount(_wei);

​        require(!isHardCapAchieved(usdAmount.sub(1)));

 

​        uint256 tokensAvailable = allocator.tokensAvailable();

​        uint256 collectedWei = contributionForwarder.weiCollected();

​        uint256 tierIndex = pricing.getTierIndex();

​        uint256 tokens;

​        uint256 tokensExcludingBonus;

​        uint256 bonus;

 

​        (tokens, tokensExcludingBonus, bonus) = pricing.getTokens(

​            _contributor, tokensAvailable, tokensSold, _wei, collectedWei

​        );

 

​        require(tokens > 0);

​        tokensSold = tokensSold.add(tokens);

​        allocator.allocate(_contributor, tokensExcludingBonus);

 

​        if (isSoftCapAchieved(usdAmount)) {

​            if (msg.value > 0) {

​                contributionForwarder.forward.value(address(this).balance)();           

​            }

​        } else {

​            // store contributor if it is not stored before

​            if (contributorsWei[_contributor] == 0) {

​                contributors.push(_contributor);

​            }

​            contributorsWei[_contributor] = contributorsWei[_contributor].add(msg.value);

​        }

 

​        usdCollected = usdCollected.add(usdAmount);

 

​        if (availableBonusAmount > 0) {     

​            if (availableBonusAmount >= bonus) {

​                availableBonusAmount -= bonus;

​            } else {

​                bonus = availableBonusAmount;

​                availableBonusAmount = 0;

​            }

​            contributorBonuses[_contributor] = contributorBonuses[_contributor].add(bonus);

​        } else {

​            bonus = 0;

​        }

 

​        crowdsaleAgent.onContribution(pricing, tierIndex, tokensExcludingBonus, bonus);

​        emit Contribution(_contributor, _wei, tokensExcludingBonus, bonus);

​    }

 

}

 

 

 

## **MAVS_17:**

### **Vulnerability_Instance_1:      out17359.sol**

​        function burn(uint _maxSrcAmount, uint _maxDestAmount, uint _minConversionRate)

​        external

​        returns(uint)

​    {

​        // ETH to convert on Kyber, by default the amount of ETH on the contract

​        // If _maxSrcAmount is defined, ethToConvert = min(balance on contract, _maxSrcAmount)

​        uint ethToConvert = address(this).balance;

​        if (_maxSrcAmount != 0 && _maxSrcAmount < ethToConvert) {

​            ethToConvert = _maxSrcAmount;

​        }

 

​        // Set maxDestAmount to MAX_UINT if not sent as parameter

​        uint maxDestAmount = _maxDestAmount != 0 ? _maxDestAmount : 2**256 - 1;

 

​        // Set minConversionRate to 1 if not sent as parameter

​        // A value of 1 will execute the trade according to market price in the time of the transaction confirmation

​        uint minConversionRate = _minConversionRate != 0 ? _minConversionRate : 1;

 

​        // Convert the ETH to ERC20

​        // erc20ToBurn is the amount of the ERC20 tokens converted by Kyber that will be burned

​        uint erc20ToBurn = kyberContract.trade.value(ethToConvert)(

​            // Source. From Kyber docs, this value denotes ETH

​            ERC20(0x00eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee),

​            

​            // Source amount

​            ethToConvert,

 

​            // Destination. Downcast BurnableErc20 to ERC20

​            ERC20(destErc20),

​            

​            // destAddress: this contract

​            this,

​            

​            // maxDestAmount

​            maxDestAmount,

​            

​            // minConversionRate 

​            minConversionRate,

​            

​            // walletId

​            0

​        );

 

​        // Burn the converted ERC20 tokens

​        destErc20.burn(erc20ToBurn);

 

​        return erc20ToBurn;

}

 

## **MAVS_18:**

### **Vulnerability_Instance_1:      out10923.sol**

  function increaseApprovalAndCall(

​    address _spender,

​    uint _addedValue,

​    bytes _data

  )

​    public

​    payable

​    returns (bool)

  {

​    require(_spender != address(this));

 

​    super.increaseApproval(_spender, _addedValue);

 

​    // solium-disable-next-line security/no-call-value

​    require(_spender.call.value(msg.value)(_data));

 

​    return true;

  }

 

 

 

 

## **MAVS_19:**

### **Vulnerability_Instance_1:    out13894.sol**

function transferAndCall(

​        address _to,

​        uint256 _value,

​        bytes _data

​    )

​    public

​    payable

​    whenNotPaused

​    returns (bool)

​    {

​        require(_to != address(this));

 

​        super.transfer(_to, _value);

 

​        // solium-disable-next-line security/no-call-value

​        require(_to.call.value(msg.value)(_data));

​        return true;

​    }

 

## **MAVS_20:**

### **Vulnerability_Instance_1:    out1874.sol**

​    function cleanBalance(address token) external returns(uint256 b) {

​        if (uint(token)==0) {

​            msg.sender.transfer(b = address(this).balance);

​            emit Clean(token, msg.sender, b);

​            return;

​        }

​        b = Yrc20(token).balanceOf(this);

​        emit Clean(token, msg.sender, b);

​        require(b>0, 'must have a balance');

​        require(Yrc20(token).transfer(msg.sender,b), 'transfer failed');

​    }

 

 

 