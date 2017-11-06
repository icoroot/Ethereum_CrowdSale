pragma solidity ^0.4.9;

contract SafeMath {
  function safeMul(uint a, uint b) internal returns (uint) {
    uint c = a * b;
    assert(a == 0 || c / a == b);
    return c;
  }

  function safeSub(uint a, uint b) internal returns (uint) {
    assert(b <= a);
    return a - b;
  }

  function safeAdd(uint a, uint b) internal returns (uint) {
    uint c = a + b;
    assert(c>=a && c>=b);
    return c;
  }

  function assert(bool assertion) internal {
    if (!assertion) throw;
  }
}

contract Token {
  function totalSupply() constant returns (uint256 supply) {}
  function balanceOf(address _owner) constant returns (uint256 balance) {}
  function transfer(address _to, uint256 _value) returns (bool success) {}
  function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {}
  function approve(address _spender, uint256 _value) returns (bool success) {}
  function allowance(address _owner, address _spender) constant returns (uint256 remaining) {}

  event Transfer(address indexed _from, address indexed _to, uint256 _value);
  event Approval(address indexed _owner, address indexed _spender, uint256 _value);

  uint public decimals;
  string public name;
}

contract StandardToken is Token {

  function transfer(address _to, uint256 _value) returns (bool success) {
    if (balances[msg.sender] >= _value && balances[_to] + _value > balances[_to]) {
      balances[msg.sender] -= _value;
      balances[_to] += _value;
      Transfer(msg.sender, _to, _value);
      return true;
    } else { return false; }
  }

  function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {
    if (balances[_from] >= _value && allowed[_from][msg.sender] >= _value && balances[_to] + _value > balances[_to]) {
      balances[_to] += _value;
      balances[_from] -= _value;
      allowed[_from][msg.sender] -= _value;
      Transfer(_from, _to, _value);
      return true;
    } else { return false; }
  }

  function balanceOf(address _owner) constant returns (uint256 balance) {
    return balances[_owner];
  }

  function approve(address _spender, uint256 _value) returns (bool success) {
    allowed[msg.sender][_spender] = _value;
    Approval(msg.sender, _spender, _value);
    return true;
  }

  function allowance(address _owner, address _spender) constant returns (uint256 remaining) {
    return allowed[_owner][_spender];
  }

  mapping(address => uint256) balances;

  mapping (address => mapping (address => uint256)) allowed;

  uint256 public totalSupply;
}

contract IcoCrowd is SafeMath {
  address public admin; //the admin address
  address public feeAddr; //the account that will receive fees
  uint public feeTradeRate;
  uint public ethHardCap;
  uint public startTime;
  uint public endTime;
  address public crowdtoken;
  uint public totalSupply;
  uint public shouldSupply;
  uint public rate;
  uint256 public constant decimals = 18;
  string public name = "GVT Sale";
  string public constant symbol = "GVT";
  mapping (address => mapping (address => uint)) private tokenList;
  event Deposit(address token, address user, uint amount, uint balance);
  event Withdraw(address token, address user, uint amount, uint balance);

  function IcoCrowd(address admin_, address feeAddr_, uint feeTradeRate_,uint ethHardCap_,address crowdtoken_,uint rate_,uint startTime_,uint endTime_) {
    admin = admin_;
    feeAddr = feeAddr_;
    feeTradeRate = feeTradeRate_;
    ethHardCap = ethHardCap_;
    startTime  = startTime_;
    crowdtoken = crowdtoken_;
    endTime = endTime_;
    rate = rate_;

  }

  function() {
    throw;
  }

  function setAdmin(address admin_) {
    if (msg.sender != admin) throw;
    admin = admin_;
  }

  function setFeeAddr(address feeAddr_) {
    if (msg.sender != admin) throw;
    feeAddr = feeAddr_;
  }

  function setFeeTradeRate(uint feeTradeRate_) {
    if (msg.sender != admin) throw;
    feeTradeRate = feeTradeRate_;
  }

  function deposit() payable {
    if (!isStart()) throw;
    uint fee = safeSub(msg.value,(msg.value / (((1 ether) + feeTradeRate)/ 10**10))* (10 ** 8));
    uint EthGive = safeSub(msg.value , fee);
    if(!(safeAdd(totalSupply,EthGive) <= ethHardCap)) throw;
    totalSupply = safeAdd(totalSupply,EthGive);
    shouldSupply = safeAdd(shouldSupply,EthGive);
    tokenList[0][msg.sender] = safeAdd(tokenList[0][msg.sender], msg.value);
    Deposit(0, msg.sender, msg.value, tokenList[0][msg.sender]);
  }

  function depositToken(uint amount) {
    if (!Token(crowdtoken).transferFrom(msg.sender, this, amount)) throw;
    tokenList[crowdtoken][admin] = safeAdd(tokenList[crowdtoken][admin], amount);
    Deposit(crowdtoken, msg.sender, amount, tokenList[crowdtoken][msg.sender]);
  }

  function withdraw(uint amount) {
    if (!(tokenList[crowdtoken][admin] == 0 && tokenList[0][feeAddr] > 0)&& !canReturn())throw;
    if (tokenList[0][msg.sender] < amount) throw;
    uint fee = safeSub(amount,(amount / (((1 ether) + feeTradeRate)/ 10**10))* (10 ** 8));
    uint EthGive = safeSub(amount , fee);
    totalSupply = safeSub(totalSupply,EthGive);
    shouldSupply = safeSub(shouldSupply,EthGive);
    tokenList[0][msg.sender] = safeSub(tokenList[0][msg.sender], amount);
    if (!msg.sender.call.value(amount)()) throw;
    Withdraw(0, msg.sender, amount, tokenList[0][msg.sender]);
  }

  function withdrawByOwner(uint amount) {
    if (msg.sender!= feeAddr && msg.sender!= admin)throw;
    tokenList[0][msg.sender] = safeSub(tokenList[0][msg.sender], amount);
    if (!msg.sender.call.value(amount)()) throw;
    Withdraw(0, msg.sender, amount, tokenList[0][msg.sender]);
  }

  function getRemain() {
    if (msg.sender != admin) throw;
    uint amount = safeSub(tokenList[crowdtoken][admin], safeMul(shouldSupply,rate));
    tokenList[crowdtoken][admin] = safeSub(tokenList[crowdtoken][admin], amount);
    if (!Token(crowdtoken).transfer(msg.sender, amount)) throw;
    Withdraw(crowdtoken, msg.sender, amount, tokenList[crowdtoken][msg.sender]);
  }

  function withdrawToken() {
    getToken();
    uint amount = tokenList[crowdtoken][msg.sender];
    tokenList[crowdtoken][msg.sender] = safeSub(tokenList[crowdtoken][msg.sender], amount);
    if (!Token(crowdtoken).transfer(msg.sender, amount)) throw;
    Withdraw(crowdtoken, msg.sender, amount, tokenList[crowdtoken][msg.sender]);
  }

  function balanceOf(address user) constant returns (uint) {
    if (tokenList[crowdtoken][admin] >= safeMul(shouldSupply,rate)){
      uint fee = safeSub(tokenList[0][user],(tokenList[0][user] / (((1 ether) + feeTradeRate)/ 10**10))* (10 ** 8));
      uint userEthAmount = safeSub(tokenList[0][user],fee);
      uint tokenAmount = safeMul(userEthAmount , rate);
      return tokenAmount;
    }else{return 0;}
  }

  function preCalcBalanceOf(address user) constant returns (uint) {
    uint fee = safeSub(tokenList[0][user],(tokenList[0][user] / (((1 ether) + feeTradeRate)/ 10**10))* (10 ** 8));
    uint userEthAmount = safeSub(tokenList[0][user],fee);
    uint tokenAmount = safeMul(userEthAmount , rate);
    return tokenAmount;
  }

  function balanceOfEth(address user) constant returns (uint) {
    return tokenList[0][user];
  }

  function totalTokenAvailable() constant returns (uint) {
    return tokenList[crowdtoken][admin];
  }

  function getToken() private{
    uint fee = safeSub(tokenList[0][msg.sender],(tokenList[0][msg.sender] / (((1 ether) + feeTradeRate)/ 10**10))* (10 ** 8));
    uint userEthAmount = safeSub(tokenList[0][msg.sender],fee);
    uint tokenAmount = safeMul(userEthAmount , rate);
    if (tokenList[crowdtoken][admin] < tokenAmount){
        tokenAmount = tokenList[crowdtoken][admin];
        userEthAmount = tokenAmount / rate;
        fee = userEthAmount * feeTradeRate;
    }
    shouldSupply = safeSub(shouldSupply,userEthAmount);
    tokenList[crowdtoken][admin] = safeSub(tokenList[crowdtoken][admin],tokenAmount);
    tokenList[crowdtoken][msg.sender] = safeAdd(tokenList[crowdtoken][msg.sender], tokenAmount);
    tokenList[0][feeAddr] = safeAdd(tokenList[0][feeAddr], fee);
    tokenList[0][admin] =   safeAdd(tokenList[0][admin], userEthAmount);
    tokenList[0][msg.sender] = safeSub(safeSub(tokenList[0][msg.sender],userEthAmount),fee);
  }

  function canReturn() constant returns(bool) {
   if (endTime > now) {
        return true;
    }
    else {
        return false;
    }
  }

  function isStart() constant returns(bool) {
   if (startTime < now) {
        return true;
    }
    else {
        return false;
    }
  }

  function after30() constant returns(bool) {
   if (startTime + 30 days < now) {
        return true;
    }
    else {
        return false;
    }
  }

  function withdrawEthAfter30(uint amount) {
    if (!after30() && msg.sender!= admin)throw;
    if (!msg.sender.call.value(amount)()) throw;
    Withdraw(0, msg.sender, amount, tokenList[0][msg.sender]);
  }

  function withdrawBalanceAfter30(uint amount) {
    if (!after30() && msg.sender!= admin)throw;
    tokenList[crowdtoken][admin] = safeSub(tokenList[crowdtoken][admin], amount);
    if (!Token(crowdtoken).transfer(msg.sender, amount)) throw;
    Withdraw(crowdtoken, msg.sender, amount, tokenList[crowdtoken][msg.sender]);
  }

  function getStatus()constant returns(uint){
    if (!isStart()){
      return  1;
    }
    if (canReturn()){
      return  2;
    }
    if (tokenList[0][feeAddr] <= 0 && totalTokenAvailable() <= 0){
      return  3;
    }
    if (tokenList[0][feeAddr] >=0 && totalTokenAvailable() >0){
      return  4;
    }
    if (tokenList[0][feeAddr] >0 && totalTokenAvailable() ==0){
      return  5;
    }
  }
}
