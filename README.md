# TO-DO-LIST-SMART-CONTRACT

Smart Contract project: To Do Notes

All projects should start with a clear specification of what we want to achieve because otherwise you’ll waste time adding unnecessary features and making things differently from the initial goal.
This project is no different. Here’s the specification that you’ll be following to create this simple Smart Contract project:
“We want to create a decentralized To Do application that allows users to store simple notes of no more than 32 letters. Each note must contain the date it was created, the address of the owner and if it’s already completed or not. Any user will be able to create notes but only the owner of each note will be able to mark them as completed.”

Let’s analyze for a second the specification to understand what we must do:
•	The notes are limited to 32 letters. This means that we’ll use bytes32 for the content of the note since it’s the most optimized way to store them.
•	The notes must contain the date they were created. We’ll use the global variable now to get the current time.
•	The notes have to contain the address of the owner that created that note. We’ll use msg.sender to get the address of that user and store it in mapping.
•	A function to mark the notes as completed. This means that we’ll have to create a struct Notes with the information required and we’ll create a function to update the state of each note.
•	Only the owner will be able to modify the notes. We’ll do this with a new type of function called modifier that allows us to make checks before doing any type of calculation. We’ll use it to compare the address of the sender to the address of the owner of the note.

Let’s start by creating a folder in the desktop called todo-dapp and then a file called TodoList.sol. This will be the contract with all the logic. In the real world you usually create a Web3.js web app to interact with the contract. We’ll do that later in this book so make sure you get this part working properly.
Inside TodoList.sol, create the base Smart Contract layout:

pragma solidity 0.4.20;
contract TodoList {}
Now you may be asking yourself. Where do I start with this application? Well, the main component of this contract is the Note element. Which is just a struct with several variables. Let’s create that:
pragma solidity 0.4.20;
contract TodoList {
   struct Todo {
      bytes32 content;
      address owner;
      bool isCompleted; 
      uint256 timestamp;
   }
}
Timestamp is the date that the note was created. The other fields are based on the specification.
We now need a way to store those notes. We could use an array or a mapping. In this case we’ll use both because each person, each address will have several notes and that can be accomplished with an array. We use a mapping because it allows us to get all the notes from a user with just his address without looping.
pragma solidity 0.4.20;
contract TodoList {
   struct Todo {
      uint256 id;
      bytes32 content;
      address owner;
      bool isCompleted;
      uint256 timestamp;
   }
   uint256 public constant maxAmountOfTodos = 100;
   
   // Owner => todos
   mapping(address => Todo[maxAmountOfTodos]) public todos;
   
   // Owner => last todo id
   mapping(address => uint256) public lastIds;
}

Note that you can make comments with a double forward slash // for line comments or with a slash with asterisk /**/ for block comments.
Here’s what I just did:
•	I’ve defined a variable called maxAmountOfTodos which is used to limit the amount of to-do notes each user can have. This is required to avoid that the number of notes that a user creates grows endlessly since we have gas limitations.
•	The mapping todos is where the notes will be stored for each user address.
•	The mapping of lastIds is just a way to keep track of the last ID used for each user, required to add new notes since we are using a fixed-size array in the todos mapping.
I like to add a comment on top of each mapping to indicate exactly what the variables inside the mapping mean for clarity purposes. It’s very important to document your code with lots of comments. It will help you understand and debug your code faster.
Keep in mind that this is not the best possible solution for this specification. It’s just my way of doing things. You could read the specification and store the notes in a single array in an efficient way. Don’t take my word. Try it yourself without copying the code and see what you can do by yourself.
At this point we can start creating the functions for this application. We’ll need a way to add notes to the mapping of todos. Here’s how I did it:
pragma solidity 0.4.20;
contract TodoList {
   struct Todo {
      uint256 id;
      bytes32 content;
      address owner;
      bool isCompleted;
      uint256 timestamp;
   }
   uint256 public constant maxAmountOfTodos = 100;
   
   // Owner => todos
   mapping(address => Todo[maxAmountOfTodos]) public todos;
   // Owner => last todo id
   mapping(address => uint256) public lastIds;
   
   modifier onlyOwner(address _owner) {
      require(msg.sender == _owner);
      _;
   }
   // Add a todo to the list
   function addTodo(bytes32 _content) public {
      Todo memory myNote = Todo(lastIds[msg.sender], _content, msg.sender, false, now);
      todos[msg.sender][lastIds[msg.sender]] = myNote;
      if(lastIds[msg.sender] >= maxAmountOfTodos) lastIds[msg.sender] = 0;
      else lastIds[msg.sender]++;
   }
}
That’s some new stuff over there. Don’t be scared. Here’s what I did:
•	I’ve created a modifier called onlyOwner to limit the access of the next function. Because we only want to allow the owner of each note to be able to modify his own notes. You’ll later see where it’s used.
•	Then I created the function addTodo with the parameter _content which is the content of the note to create. Inside this function, I’m creating a memory note called myNote and then I’m adding that note to the array of notes of that user, in the mapping todos.
•	Finally, I’m updating the lastId of that user from the lastIds mapping to be able to add new notes since you need to know the index of each element inside the fixed-size array of todos.
The parameters of the functions in Solidity usually have an underscore _ in front of them. This is to avoid problems with variables using the same name. For instance:
bytes32 content;
function addTodo(bytes32 content) {}
Notice that those 2 content variables use the same name. This won’t work because in Solidity you can’t use already existing names inside the function. So, we always add an underscore in the function parameters to avoid this problem.
Take a look at the struct instance myNote. Do you see something unusual? If you said “The keyword memory is new, I don’t know what’s that, please explain” you are correct. Memory is a special word that you can use before the variable name when you declare the variable.
It indicates that you don’t want to store that variable on the blockchain. It keeps the variable in the memory of the user’s computer executing that code and it’s deleted after the function execution. We must do this on the struct instance Todo because without the memory keyword, Solidity tries to declare variables in storage.
This means that when you create a new struct instance, Solidity tries to store it in the storage which is permanently writing information on the blockchain. The Ethereum Virtual Machine has three areas where it can store items.
1.	The first is “storage”, where all the contract state variables reside. Every contract has its own storage and it is persistent between function calls and quite expensive to use.
2.	The second is “memory”, this is used to hold temporary values. It is erased between (external) function calls and is cheaper to use.
3.	The third one is the stack, which is used to hold small local variables. It is almost free to use, but can only hold a limited amount of values.
For almost all types, you cannot specify where they should be stored, because they are copied every time they are used.
The types where the so-called storage location is important are structs and arrays. If you e.g. pass such variables in function calls, their data is not copied if it can stay in memory or stay in storage. This means that you can modify their content in the called function and these modifications will still be visible in the caller.
In summary: you have to add the keyword “memory” to temporary structs and arrays inside functions for avoiding problems with the blockchain storage.
I highly recommend you to use the Remix IDE for developing Smart Contracts. It’s a web app where you can write contracts and see errors immediately. You can access it on Remix.ethereum.org. Then, search for “Remix Ethereum tutorial” on youtube to understand how it works. Later in this book you’ll see how it’s used to deploy Smart Contracts on the Ropsten and main Ethereum blockchain networks.
All the Smart Contract code that I write ends up in that IDE because it helps me spot errors quickly. It also allows you to execute the functions of your Smart Contract in the real Ethereum blockchain. You’ll need Metamask which is an extension for browsers that allow you to connect to the blockchain.
Before continuing with this I want you to do these 2 tasks:
•	Learn how to use the Remix code editor by looking at videos on youtube. You can use it for free at http://Remix.ethereum.org
•	Install Metamask and learn how to use it by searching videos. You can install it by going to Metamask.io.
Those tools are a must to any Ethereum developer.
After that we can continue with the project on the Remix IDE. The next step is to mark to-dos as completed. We’ll do that with a function:
pragma solidity 0.4.20;
contract TodoList {
   struct Todo {
      uint256 id;
      bytes32 content;
      address owner;
      bool isCompleted;
      uint256 timestamp;
   }
   uint256 public constant maxAmountOfTodos = 100;
   
   // Owner => todos
   mapping(address => Todo[maxAmountOfTodos]) public todos;
   // Owner => last todo id
   mapping(address => uint256) public lastIds;
   // Add a todo to the list
   function addTodo(bytes32 _content) public {
      Todo memory myNote = Todo(lastIds[msg.sender], _content, msg.sender, false, now);
      todos[msg.sender][lastIds[msg.sender]] = myNote;
   
      if(lastIds[msg.sender] >= maxAmountOfTodos) lastIds[msg.sender] = 0;
      else lastIds[msg.sender]++;
   }
   // Mark a todo as completed
   function markTodoAsCompleted(uint256 _todoId) public {
      require(_todoId < maxAmountOfTodos);
      require(!todos[msg.sender][_todoId].isCompleted);
      todos[msg.sender][_todoId].isCompleted = true;
   }
}
Here’s what I just did:
•	I added the function markTodoAsCompleted which will allow us to mark a 
•	to-do as completed whenever the user wants. I simply get the specific to-do from the _todoId sent by the user of the application to mark it as completed.
•	You see that I’m using a new global function called require. This is a very important function that allows us to check some conditions before executing that function. In this case I’m saying: “Require that the to-do id sent is smaller than the maximum number of to-dos” and “Require that the to-do selected is not completed”.
•	Most of the require statements are used at the top of the function for checking the parameters of the function even though you can use the require function wherever you want. I’m using the exclamation sign ! in front of the boolean value to negate and invert the condition.
•	If require results true, the function executes. If require results in false, the function execution is stopped, and the user will receive an error. The remaining gas will be refunded to the user.
•	Finally, I set the selected to-do as completed.
When a user executes a function from the Smart Contract, he needs to send some amount of gas. The minimum amount of gas required to execute any function inside a Smart Contract is 21000 and can be up to 8 million. It’s just a fee that gets converted to ether that miners receive for processing the transaction, in this case, a function execution in your Smart Contract.
In Solidity you can use for and while loops:
A for loop in Solidity is limited by the gas that you send to the Smart Contract. It will continue looping until it loops all the items or until it runs out of gas. Because in each iteration it’s consuming gas for processing that code. In essence, this means that your for loops are limited to a specific number of loops, you can’t loop endlessly. Here’s the syntax for a for loop:
for(uint256 i = 0; i < myArray.length; i++) {
   // Do something with the array
}
That’s why for and while loops are not recommended in Solidity because you’ll probably run into gas problems.
It’s your job to optimize the code and to make sure that the loops are not breaking the gas limits by limiting the arrays like I did in this case.
You may have noticed that the function markTodoAsCompleted can be executed by anyone that knows the address of this Smart Contract once it’s deployed. By default, all functions are public. This means that anyone that has the contract code can execute and see all the functions.
However, we don’t want that. In this case we want to only allow the to-do owner to modify his own notes. That’s the main reason we have an owner parameter in the to-do struct. Luckily, we can do that in Solidity with a very powerful component called modifiers. Let’s see how it will be used in this project:
pragma solidity 0.4.20;
contract TodoList {
   struct Todo {
      uint256 id;
      bytes32 content;
      address owner;
      bool isCompleted;
      uint256 timestamp;
   }
   uint256 public constant maxAmountOfTodos = 100;
   // Owner => todos
   mapping(address => Todo[maxAmountOfTodos]) public todos;
   // Owner => last todo id
   mapping(address => uint256) public lastIds;
   modifier onlyOwner(address _owner) {
      require(msg.sender == _owner);
      _;
   }
   // Add a todo to the list
   function addTodo(bytes32 _content) public {
      Todo memory myNote = Todo(lastIds[msg.sender], _content, msg.sender, false, now);
      todos[msg.sender][lastIds[msg.sender]] = myNote;
      if(lastIds[msg.sender] >= maxAmountOfTodos) lastIds[msg.sender] = 0;
      else lastIds[msg.sender]++;
   }
   // Mark a todo as completed
   function markTodoAsCompleted(uint256 _todoId) public onlyOwner(todos[msg.sender][_todoId].owner) {
      require(_todoId < maxAmountOfTodos);
      require(!todos[msg.sender][_todoId].isCompleted);
      todos[msg.sender][_todoId].isCompleted = true;
   }
}
You can see that I created a modifier called onlyOwner. A modifier is like a function that gets execute before the actual function. You first define the modifier, which usually has a require statement inside, then you add the underscore _; which indicates that the code will be inserted below that underscore if you pass the require checks successfully.
Finally, you have to add that modifier to the function that you want to be affected. The modifier will be executed before the actual function every time. You have to write it after the name of the function, before the opening curly bracket of the function affected {.
Are you still following me? Great.
In this case I’m creating the modifier onlyOwner, which receives an address and checks if the user that executed that modifier is the owner that you sent him. Then I added the modifier to the function markTodoAsCompleted to only allow the modification of that note by the owner of it.
Yes, it’s a lot of information but you’ll see how powerful the modifiers are. You’ll get used to them pretty quickly as you write Smart Contracts. The structure of a modifier usually is:
modifier <name>(<parameters>) {
   require(<something>);
   _;
}
You can also add parameters to the modifiers. Otherwise, you have to remove the brackets (<parameters>).
Now the function markTodoAsCompleted is only executable by the owner of the note selected by the id. Your private notes are safe now!
Here’s what you accomplished so far:
1.	You created a contract called TodoLists whose purpose is to create and modify to-do notes.
2.	Then you defined the structure of an individual To-do and you created the mapping of all the notes for each user.
3.	Then you created a function to add new to-dos to the sender address. Also, you added a function to mark the to-dos as completed.
4.	Finally, you’ve restricted the access of the to-dos to only allow modification of the to-dos a user owns with the modifier only owner.


