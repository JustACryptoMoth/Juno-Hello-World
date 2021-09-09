# Juno-Hello-World

Dependencies:

Any client that wants to use the below actions MUST have their Keplr wallet connected to the app. 
If the Keplr wallet is not connected, the buttons that allow for these actions will not appear and they cannot be submitted. 

User Actions:

- Connect Keplr wallet
  - JavaScript Side:
    - This is the default/first question that is asked of the user. The webpage will be blank except for the logos and 
      the prompt to ask the user to connect their Keplr wallet.
    - When user hits connect, JavaScript code will connect this session with the user's Keplr wallet.
    - This looks like sample JS code to connect to Keplr: https://github.com/ebaker/next-cosmwasm-keplr-starter/blob/5b6d0437c5097faef90ccacfa502e14a65d82c00/services/keplr.tsx#L11

- View polls
  - JavaScript Side:
    - If user has connected Keplr wallet, this is the default screen that will be shown.
    - The most recent X polls can have their IDs retrieved by calling the smart contract's get_recent_polls(X) method.
    - The data for each pollID can be pulled by calling the smart contract methods:
      - get_poll_type(pollID)
      - get_poll_question(pollID)
      - get_poll_question_upvotes_downvotes(pollID)    // to display graphically next to upvote/downvote buttons
      - get_poll_answer_choices(pollID)
    - Next to each poll is a pie graph of the poll's results. The voted answers can be pulled by the smart contract method:
      - get_poll_voted_answers(pollID)

- Upvote poll
  - JavaScript Side:
    - Each poll has an up arrow and down arrow next to the poll on the left (similar to StackOverflow)
    - The user can upvote or downvote a poll. This represents the "popularity" of the poll.
    - Either above or inside of the arrows should be the "total poll score", which is upvotes minus downvotes.
    - The upvote or downvote button will show as highlighted if the user has already voted on the poll
    - If a user clicks either the upvote or downvote buttons, it will signal a transaction to the smart contract.
    - If the user has already upvoted or downvoted a poll, the only new upvote or downvote that differs from previous upvote or downvote is accepted.
    - The user will be prompted with the Keplr prompt to sign the transaction of upvote or downvote.
    - Once the transaction goes through asynchronously, the poll value will update to the new poll's value
  - Smart Contract side: 
    - When the user submits the transaction on a poll, 

- Create simple poll
  - JavaScript Side:
    - User selects "create simple poll" button
    Popup appears with the following information:
      - Test input field for the question to be asked in the poll
      - Yes/No button if user wants to ask anonymously
      - Text containing the default answer choices for the poll will be {Yes, No, No with Veto, Abstain}
      - Two buttons at bottom of popup: {Submit Poll, Cancel}
    
    - 
  - Smart Contract side:
    - generates random ID
    - assign creator's address to [creator], unless specified anonymous then None
    - creates [Question] (cannot be changed), initializes [QuestionUpvotes/QuestionDownvotes] to 0
    - creates [Options] (cannot be changed), initializes [OptionsVotes] to all be 0
    - add Poll to [Polls]


- Create multi poll
    - generates random ID
    - assign creator's address to [creator], unless specified anonymous then None
    - creates [Question] (cannot be changed), initializes [QuestionUpvotes/QuestionDownvotes] to 0
    - creates [Options] (cannot be changed), initializes [OptionsVotes] to all be 0
    - add Poll to [Polls]

// Structure of the polls represented on blockchain:

enum PollType {
  Simple,
  Multi,
}

Each poll is a struct:

trait Poll {
  ID: u64,
  Creator: Option<Address>,
  Question: String,
  QuestionUpvotes: Mutable u32,
  QuestionDownvotes: Mutable u32,
  Type: PollType
}

trait Vote {}

struct SimpleVote is Vote {
  Yes,
  No,
  NoWithVeto,
  Abstain,
}

struct MultiVote is Vote {
  A,
  B,
  C,
  D,
  E,
}

struct SimplePoll is Poll {
  VoterVotes: HashMap<Address, SimpleVote>
  Type: PollType = Simple,
}

struct MultiPoll is Poll {
  Options: Vector<String>,
  OptionsVotes: HashMap<Address, Vector<MultiVote>>,
  Type: PollType = Multi,
}

//////////////////////
// Other variables: //
//////////////////////

Polls: Vector<Poll>
Poll_Askers: Vector<Address>
Voters: Vector<Address>

//////////////
//          //
// Methods: //
//          //
//////////////

/////////////////////////////////
// Methods that fetch poll IDs //
/////////////////////////////////

// Fetches _num_polls most recent created polls
fn get_recent_polls(_num_polls: u32) -> Vector<u64> 

// Fetches _num_polls most popular polls - polls with the highest value of QuestionUpvotes + QuestionDownvotes.
fn get_interesting_polls(_num_polls: u32) -> Vector<u64>

// Fetches _num_polls most recent created polls, and then ordered by popularity with highest (QuestionUpvotes+QuestionDownvotes) polls at the top. 
fn get_recent_interesting_polls(_num_polls: u32) -> Vector<u64>


/////////////////////////////////////////
// Methods that fetch data from polls: //
/////////////////////////////////////////

// Returns the poll type (Simple = always asks Yes / No / No with Veto / Abstain with 1 choice each. 
// Multi allows custom responses with multiple possible responses.)
fn get_poll_type(_poll_id: u64) -> String

// Returns the question of the poll with provided _poll_id.
fn get_poll_question(_poll_id: u64) -> String

// Returns a tuple of (upvotes, downvotes) of the poll with provided _poll_id. Returns None if 
fn get_poll_question_upvotes_downvotes(_poll_id: u64) -> Result<u32, u32>

// Returns the possible (string) answers to be selected by a voter in the poll
fn get_poll_answer_choices(_poll_id: u64) -> Vector<String>

// Returns the amount of votes of each (string) answer for a given poll
fn get_poll_voted_answers(_poll_id: u64) -> HashMap<String, u32>

////////////////////////////////////
// Methods that modify poll data: //
////////////////////////////////////

// User (self) upvotes the poll. Increments the QuestionUpvotes value within the poll.
// Result is error if unsuccessful, or value of new upvotes if successful.
fn upvote_poll(_poll_id: u64) -> Result

// User (self) downvotes the poll. Increments the QuestionDownvotes value within the poll.
// Result is error if unsuccessful, or value of new downvotes if successful.
fn downvote_poll(_poll_id: u64) -> Result

// Returns the poll type (Simple = always asks Yes / No / No with Veto / Abstain with 1 choice each. 
// Multi allows custom responses with multiple possible responses.)
fn get_poll_type(_poll_id: u64) -> String

//////////////////////////////////////////////////////////////
// Methods that return addresses of poll creators & voters: //
//////////////////////////////////////////////////////////////

// Returns all addresses that have voted on a specific poll.
fn get_voters(_poll_id: u64) -> Vector<Address>

// Returns all addresses that have ever voted on any poll.
fn get_all_voters() -> Vector<Address>

// Returns the asker/creator of a poll. Option evaluates to None if asker/creator requested to remain anonymous when creating the poll. 
fn get_poll_creator(_poll_id: u64) -> Option<Address>

// Returns all addresses that have ever created a poll.
fn get_all_poll_creators() -> Vector<Option<Address>>

// Tips the question asker/creator of a poll amt of Juno. Does not go through if 0 is tipped. Does not go through if question asker asked anonymously.
fn tip_poll_creator(_amt: u64, _poll_id: u64) -> Result



