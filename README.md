# CodeAlpha_Voting-System-Smart-Contract
Source Code- 

    // SPDX-License-Identifier: MIT
     pragma solidity ^0.8.0;
    contract PollingSystem {
    struct Poll {
        string title;
        string[] options;
        uint endTime;
        mapping(uint => uint) voteCounts; 
        mapping(address => bool) hasVoted; 
        bool exists;
    }
    
    uint public pollCount;
    mapping(uint => Poll) public polls;
    event PollCreated(uint pollId, string title, uint endTime);
    event VoteCast(uint pollId, address voter, uint optionIndex);

    function createPoll(string memory _title, string[] memory _options, uint _durationInMinutes) public {
        require(_options.length >= 2, "Poll must have at least 2 options");

        Poll storage newPoll = polls[pollCount];
        newPoll.title = _title;
        newPoll.options = _options;
        newPoll.endTime = block.timestamp + (_durationInMinutes * 1 minutes);
        newPoll.exists = true;

        emit PollCreated(pollCount, _title, newPoll.endTime);
        pollCount++;
    }

   
    function vote(uint _pollId, uint _optionIndex) public {
        require(polls[_pollId].exists, "Poll does not exist");
        Poll storage poll = polls[_pollId];
        require(block.timestamp < poll.endTime, "Voting has ended");
        require(!poll.hasVoted[msg.sender], "You have already voted");
        require(_optionIndex < poll.options.length, "Invalid option");

        poll.voteCounts[_optionIndex]++;
        poll.hasVoted[msg.sender] = true;

        emit VoteCast(_pollId, msg.sender, _optionIndex);
    }
    function getVotes(uint _pollId, uint _optionIndex) public view returns (uint) {
        require(polls[_pollId].exists, "Poll does not exist");
        return polls[_pollId].voteCounts[_optionIndex];
    }
    function getPollOptions(uint _pollId) public view returns (string[] memory) {
        require(polls[_pollId].exists, "Poll does not exist");
        return polls[_pollId].options;
    }
    function getWinningOption(uint _pollId) public view returns (uint winningOptionIndex) {
        require(polls[_pollId].exists, "Poll does not exist");
        require(block.timestamp >= polls[_pollId].endTime, "Poll still active");

        Poll storage poll = polls[_pollId];
        uint highestVotes = 0;

        for (uint i = 0; i < poll.options.length; i++) {
            if (poll.voteCounts[i] > highestVotes) {
                highestVotes = poll.voteCounts[i];
                winningOptionIndex = i;
            }
         }
      }
    }
