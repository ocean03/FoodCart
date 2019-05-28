Truffle Debugger
Truffle Debugger is a command line tool integrated into the Truffle Framework which allows smart contract transactions to be debugged using the smart contract code and transaction artifacts. When the debugger is started, the command line interface provides a list of addresses transacted against or created during the course of the transactions, an initial entry point for the transaction and a list of available commands for using the debugger. Some of the debugger commands are highlighted below:


[o] step over:

This command evaluates the instructions evaluated by the virtual machine relative to the contract until the next line in the current smart contract file, skipping the functions referenced at the current execution point(if any).

[i] step into:

This command steps into the function called at the current execution point.

[u] step out:

This command will have the debugger step out of the current function, to the next line executed right after it.

[n] step next:

This command steps to the next logical statement or expression in the current function called.

[;] step instruction:

This command steps through each individual instruction evaluated by the virtual machine.

You can check out other commands here

Thus far, we have talked about debugging, and the Truffle Debugger, let’s have an example to really drive this home.

Debugging a FoodCart smart contract
The smart contract below models a simple food cart which allows anyone to add food items to the cart for sale and also allows anyone with enough ether to buy a food item off the cart. This contract is just for the purpose of illustration, and it should not be used in a production environment. Before we start this example, the following are required to follow this example without any issues:

Truffle 4.0 or above
Solidity compiler version 0.4.24 or above
Private blockchain (Ganache CLI v6.1.6 or above)
The repository for this article can be found here, just in case you want to follow through the debugging process without coding the smart contract.

Step 1: Building the smart contract
clone this repositori and understand the code written inside the contract.
Before we proceed any further, let me explain the smart contract above. The FoodCart smart contract can be sectioned into 6 parts:

The state variables: The state variables, owner and skuCount store the address of the owner and the count of food items added to the FoodCart respectively. The mapping foodItems maps skus to food items.
The enum: The State enum is a user-defined data type that holds the state of the food items on the cart. The types listed in the enum are explicitly convertible to and from integers, i.e (ForSale = 0, Sold = 1).
The events: The events ForSale and Sold log the details of the food items that are put on sale or sold. These events can be called by callbacks in javascript and their contents can be used to make using dApps more interactive.
The struct: The struct FoodItem is a user-defined type which holds the properties of a food item. These properties can be accessed using a dot notation on the struct.
The function modifiers: The modifiers doesFoodItemExist, isFoodItemForSale, and hasBuyerPaidEnough are functions that automatically check a condition prior to executing the function on which they are applied to. The roles of the function modifiers used in this smart contract are obvious from their names.
The functions: The functions addFoodItem, buyFoodItem and fetchFoodItem adds food items to the contract, allows a food item to be bought and allows the details of food item to be viewed respectively. The constructor function initializes the owner state variable with the address that the contract is deployed on. The anonymous payable function allows ether to the sent to this contract.
Step 2: Deploying the smart contract
Let’s deploy the smart contract on the private network.

In your IDE, create a new file called 2_foodcart_migration.js in the migrations folder. Open this file after creating it and add this code:

This piece of code will enable the framework to deploy our FoodCart.sol contract to the private blockchain.

Open your terminal and navigate into the folder for this project and type this command:
truffle develop
This command starts a development blockchain with which we can start testing our contract. Running that command should produce this result:


The image above shows that a development blockchain has been started on port 9545, test accounts have also been created for use, and the terminal is navigated to a new prompt truffle(develop)>.

In the truffle(develop)> prompt, type the compile command to compile the contract. The result of the compilation is stored in the build folder of your project directory.
truffle(develop)> compile
Running the compile command should produce this result:


Finally, let’s migrate the compiled contract to the blockchain for deployment. At the terminal, type the migrate command to migrate the contract to the development blockchain that has been started for us.
truffle(develop)> migrate
Running the migrate command should produce this result:


Step 3: Interacting with the smart contract
Let’s interact with our smart contract and get a feel of how it works. We will be adding food items to the food cart, checking the details of food items added to the cart, and also buy food items off the cart with Ether from one of the accounts created for us when we started the development blockchain.

Still in the truffle(develop)> prompt, create a foodCart variable and store the instance of the deployed contract in it.
truffle(develop)> let foodCart; truffle(develop)> FoodCart.deployed().then((instance) => { foodCart = instance; });
In the short code snippet above, we access the deployed FoodCart contract via the web3 .deployed method which returns a promise interface and we pass a function which stores the instance of the deployed contract in the foodCart variable. With the instance of the contract stored in the foodCart variable, we can access the functions of the contract through the foodCart variable.

Let’s add some food items to the food cart by running the following code snippets at the truffle(develop)> prompt. First, let’s add a function that will help us add food items and print the result of the transaction at the console easily.

The addFoodItemToCart function allows us to easily add food items to the cart by passing the name and price of the food item to the function as arguments when it is called. The function acts as a wrapper for the addFoodItem in the smart contract by making the smart contract calls for us and formating the log output returned from the function call. We can add some food items with the addFoodItemToCart function by running this commands:


Now that we have about 3 food items on the the food cart, let’s buy some food items off the cart. Before we can buy a food item, we need to specify what address we will be using to make our purchase, because it is from this address that we can get the funds in ether to buy food items. Recall that when we started our private blockchain, 10 test accounts were created for us and each of these accounts was credited with 100 ether which we can use to buy anything we want, we are rich 🤑🤑. Let’s create a variable to store one of these addresses or accounts so that we can easily call it anytime we want.
truffle(develop)> const buyerAddress = web3.eth.accounts[1]; truffle(develop)> buyerAddress '0xf17f52151ebef6c7334fad080c5704d77216b732'
Since we have an account 0xf17f52151ebef6c7334fad080c5704d77216b732 (yours will be different), we can add a function to buy food items off the food cart at our truffle(develop)> prompt.


The buyFoodItemFromCart allows us to buy food items off the cart by calling the function with the itemSku and amount of the item. The amount to be paid for the item is taken from the account that was stored in our buyerAddress variable. Let’s buy Fried Rice off the food cart. From our previous transactions, we recall that the sku for Fried Rice is 0 and it’s price is 10 wei. So at the truffle(develop)> prompt, we call the buyFoodItemFromCart function with these values:


From the output, we see that the state of the item is Sold and the foodItemExist is now false, indicating that this item is no longer for sale.

Step 4: Debugging errors in the smart contract
If you have made it thus far, you deserve a Noble prize for Tenacity 🎖, enjoy the fame it brings. So far, we have seen how the contract should behave. To be able to use the Truffle Debugger feature, we would introduce some errors while interacting with the contract and then use the debugger to debug the error and get it fixed. To debug a transaction, we need to have the hash of the transaction and then run command debug [transaction hash] at the truffle(develop)> prompt followed by any of the debugger commands until we find out where the transaction failed.

We will try the following invalid transactions:

Try to buy a food item that does not exist
Try to pay an amount less than the price of a food item
Before we start these transactions, we need to start the truffle develop logger in another terminal, but still in the sample project. Open a new terminal, navigate to the FoodCart project and run this command truffle develop --log.

FoodCart $ truffle develop --log
You should get this output:

Connected to existing Truffle Develop session at http://127.0.0.1:9545/
The logger connects to an existing session at the port specified, listens for transaction events, and logs the output of the transaction which should include items like transaction hash, block number, etc. Let’s go ahead and make the life of our contract a living hell 👹.

Invalid Transaction #1: Try to buy a food item that does not exist
From our previous transaction where we added food items to cart, we had skus 0, 1, 2 respectively for the 3 food items. We will try to use the sku 6 to buy a food item that does not exist.


Trying to buy the item as shown in the command above produces the error UnhandledPromiseRejectionWarning: Error: VM Exception while processing transaction: revert ... which does not really tell us what or where the problem came from. Now, this can be a real headache when there’s no debugger. I have spent many hours trying to debug a smart contract without using the debugger and it can drain the last blood out of one. Let’s have a look at log output and see what we have there.


The most important content of the log to us is the transaction hash 0x90a2ef83e54426848523a4edcf0c9628045e03c69d95f989c5b3b557367c848c, this will be different from what you have on your machine. With the transaction hash, we can debug the transaction and figure out where the problem is. Let’s debug this transaction. Copy the hash of the transaction from your debugger terminal and run this command: debug your-trasanction-hash at the truffle(develop)> prompt. Your output should look like this:


From the image, we see that when the debug command is run, the debugger compiles the contracts, gathers transaction data, fetches the address that was affected and the contract that was deployed at that address, and shows a list of debugger commands that we can use to interact with the debugger. The most interactive command to use at the debugger is the step next command which steps through all the instructions executed during a transaction one at a time. The step next command is executed by pressing enter or n, at the keyboard of course.

At the debug(develop:0x90a2ef83...)> prompt, press enter continuously, following each instruction step until we reach the instruction where the transaction failed. There are 8 steps before we reach the final instruction that halted our transaction - you will have to press enter 9 times to reach the halting instruction. Let’s look at the first 4 steps from this image below:


From the image, we see that at line 61, the buyFoodItem function was called, but before the execution of the function, the modifier doesFoodItemExist at line 64 was called to check if the food item to be bought exists in the cart. On further inspection, on line 33, we see that the modifier uses the require function to ensure that the foodItemExist property of the food item with the sku given is true. Let’s check what happened in the rest of the steps in the image below:


At last step on line 33 in the image above, we see that the transaction halted because the transaction failed the required condition to be able to buy a food item from the cart. The item that we wanted to buy does not exist. The require function throws a state-reverting exception when a condition to which it is applied to fails. A state-reverting exception reverts or undoes all changes made to the state in the current call (and all its sub-calls) and also flag an error to the caller. We have been able to successfully debug this code using the Truffle Debugger, and it has been pretty amazing.

Invalid Transaction #2: Try to pay an amount less than the price of a food item
In this transaction, we will try to buy a food item with less amount than the price of the food item. In ‘step 3: Interacting with the smart contract’, we added 3 food items. Let’s try to buy ‘Chicken Pepper Soup’ with sku 1 and price 10 wei. At the truffle(develop)> prompt, call the buyFoodItem function to buy a food item with less amount than the price.

truffle(develop)> buyFoodItemFromCart(1, 5); 
truffle(develop)> (node:40269) UnhandledPromiseRejectionWarning: Error: VM Exception while processing transaction: revert
Expectedly, this transaction fails, giving a cryptic error message as usual. To debug this transaction, we go to our log terminal and copy the hash of the transaction. On my machine the transaction hash is 0x5dc12d3ac524cfd1e3c0ea9265a155149ff16fcb18f904016a87e83cca5f9a30. With this transaction hash, we shall debug the transaction and figure out where the transaction halted.

truffle(develop)> debug 0x2ad4c056a678f221d761d0bc11f641f6b4e63b3016587acd8b16ccf5295bd4d3
Like we did in the first debugging process, we will use the step next debugger command to step through the transactions as it is very interactive and shows all the instructions that have been executed until we reach the instruction that halted the transaction. Use enter to step through the transaction history until you reach the instruction that halted the transaction as shown in the image below:


In the image above, we see that at line 44, the modifier hasBuyerPaidEnough is called before the buyFoodItemFromCart function is executed, and at line 45, the modifier requires that value sent by the buyer is greater than or equal to the price of the item. However, we know that we flouted this condition hence, it halted our transaction. So if we send the right amount, we will surely buy the food item we need.

Conclusion
In conclusion, the importance of using a debugger while building smart contracts cannot be overemphasized. With a debugger, we can test our modifiers to ensure that our contract behaves appropriately, we can ensure that our contracts are secure by sending malicious transactions to our contract to see how it will behave and properly debug the transaction to see the instructions that have been executed up until the transaction halted, if it did. Most importantly, we can save our most valuable asset which is time, by using a debugger whenever we face issues with our smart contracts, instead of fumbling around with our code.
