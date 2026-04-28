# Yield-Farming-Staking-Pool
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract YieldStakingPool {
    mapping(address => uint256) public userStaked;
    mapping(address => uint256) public userRewardDebt;

    uint256 public totalStaked;
    uint256 public accRewardPerShare;
    uint256 public lastRewardTime;
    uint256 public rewardRate; // rewards per second

    address public rewardToken; // In real use, this would be an ERC20

    event Staked(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);
    event RewardClaimed(address indexed user, uint256 reward);

    constructor(uint256 _rewardRate) {
        rewardRate = _rewardRate;
        lastRewardTime = block.timestamp;
    }

    function updatePool() internal {
        if (totalStaked == 0) {
            lastRewardTime = block.timestamp;
            return;
        }
        uint256 timePassed = block.timestamp - lastRewardTime;
        uint256 newReward = timePassed * rewardRate;
        accRewardPerShare += (newReward * 1e18) / totalStaked;
        lastRewardTime = block.timestamp;
    }

    function stake() public payable {
        updatePool();
        if (userStaked[msg.sender] > 0) {
            uint256 pending = (userStaked[msg.sender] * accRewardPerShare / 1e18) - userRewardDebt[msg.sender];
            if (pending > 0) _sendReward(pending);
        }
        userStaked[msg.sender] += msg.value;
        totalStaked += msg.value;
        userRewardDebt[msg.sender] = userStaked[msg.sender] * accRewardPerShare / 1e18;
        emit Staked(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) public {
        updatePool();
        uint256 pending = (userStaked[msg.sender] * accRewardPerShare / 1e18) - userRewardDebt[msg.sender];
        if (pending > 0) _sendReward(pending);

        userStaked[msg.sender] -= amount;
        totalStaked -= amount;
        userRewardDebt[msg.sender] = userStaked[msg.sender] * accRewardPerShare / 1e18;

        payable(msg.sender).transfer(amount);
        emit Withdrawn(msg.sender, amount);
    }

    function _sendReward(uint256 amount) internal {
        // In production, transfer ERC20 reward token
        payable(msg.sender).transfer(amount); // Simplified
        emit RewardClaimed(msg.sender, amount);
    }
}
