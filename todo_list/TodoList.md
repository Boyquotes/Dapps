## Exemple - Todo List  
### Smart contract à utiliser 
```js
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.8.0;

contract TodoList {
    struct Task {
        uint id;
        string content;
        bool completed;
    }

    uint public taskCount = 0;
    mapping(uint => Task) public tasks;

    event TaskCreated(
        uint id,
        string content,
        bool completed
    );

    event TaskCompleted(
        uint id,
        bool completed
    );

    constructor() public {
        createTask("First task ...");
    }

    function createTask(string memory _content) public {
        taskCount ++;
        tasks[taskCount] = Task(taskCount, _content, false);
        
        emit TaskCreated(taskCount, _content, false);
    }

    function toggleCompleted(uint _id) public {
        Task memory _task = tasks[_id];
        
        _task.completed = !_task.completed;
        tasks[_id] = _task;
        
        emit TaskCompleted(_id, _task.completed);
    }
}
```
### Création du projet et installation des dépendances

Pour interagir avec la blockchain Ethereum à partir de JavaScript, nous aurons besoin de la librairie web3. 

 **1. Création d'un nouveau projet**
 **2. Création d'un fichier *server.js***
```js
var  express  =  require('express');
var  app  =  express();

app.use(express.static(__dirname  +  '/public')); 

var  port  =  8000; // you can use any port

app.listen(port);
console.log('server on'  +  port);
```
 **3. Création d'un dossier public qui va contenir notre fichier html par la suite** 
```console
 $ mkdir public 
 ```

 **4. Création d'un nouveau fichier *index.html* (sous le dossier public) qui contiendra le code suivant :** 
```html
<!DOCTYPE html>

<html>
<head>

<meta charset='utf-8'>
<meta http-equiv='X-UA-Compatible' content='IE=edge'>
<meta name='viewport' content='width=device-width, initial-scale=1'>
<title>TODO List Dapp</title>
<script type="text/javascript" src="https://unpkg.com/web3@1.3.1/dist/web3.min.js"></script>
<script src="https://unpkg.com/@metamask/detect-provider/dist/detect-provider.min.js"></script>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.0/jquery.min.js"></script>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-giJF6kkoqNQ00vy+HMDP7azOuL0xtbfIcaT9wjKHr8RbDVddVHyTfAAsrekwKmP1" crossorigin="anonymous">

<style>
	main {
		margin-top: 60px;
	}

	#content {
		display: none;
	}

	form {
		width: 350px;
		margin-bottom: 10px;
	}

	ul {
		margin-bottom: 0px;
	}

	#completedTaskList  .content {
		color: grey;
		text-decoration: line-through;
	}
</style>

</head>
<body>
	<nav class="navbar navbar-dark bg-dark">
		<ul class="container-md">
			<li>
				<a class="navbar-brand col-sm-3 col-md-2 mr-0">Todo List Dapp</a>
			</li>
			<li>
				<small><a class="navbar-text"><span id="account"></span></a></small>
			</li>
		</ul>
	</nav>
	<br>
	<div class="container-fluid">
		<div class="row">
			<main role="main" class="col-lg-12 d-flex justify-content-center">
				<div id="loader" class="text-center">
					<p class="text-center">Loading...</p>
				</div>
				<div id="content">
					<form onSubmit="createTask(); return false;">
						<input id="newTask" type="text" class="form-control" placeholder="Add task ..." required>
						<input type="submit" hidden="">
					</form>
					<ul id="taskList" class="list-unstyled">
						<div class="taskTemplate" class="checkbox" style="display: none">
							<label>
								<input type="checkbox"  />
								<span class="content">Task content goes here ...</span>
							</label>
						</div>
					</ul>
					<ul id="completedTaskList" class="list-unstyled">
					</ul>
				</div>
			</main>
		</div>
	</div>
</body>
</html>
```
 **5. Ajout de la partie script :** 
 ```js
 async  function  load() {
	await  loadWeb3();
	window.contract  =  await  loadContract();
	window.account  =  await  getCurrentAccount();
	await  render();
}

async  function  loadWeb3() {
	const  provider  =  await  detectEthereumProvider()

	if (provider) {
		// handle provider
		window.web3  =  new  Web3(provider);
		window.loading  =  false;
		
	} else {
		// handle no provider
		console.log("no provider");
	}
}

async  function  loadContract() {
	return  await  new  window.web3.eth.Contract([
		{
		"inputs": [],
		"stateMutability": "nonpayable",
		"type": "constructor"
		},
		{
		"anonymous": false,
		"inputs": [
		{
		"indexed": false,
		"internalType": "uint256",
		"name": "id",
		"type": "uint256"
		},
		{
		"indexed": false,
		"internalType": "bool",
		"name": "completed",
		"type": "bool"
		}
		],
		"name": "TaskCompleted",
		"type": "event"
		},
		{
		"anonymous": false,
		"inputs": [
		{
		"indexed": false,
		"internalType": "uint256",
		"name": "id",
		"type": "uint256"
		},
		{
		"indexed": false,
		"internalType": "string",
		"name": "content",
		"type": "string"
		},
		{
		"indexed": false,
		"internalType": "bool",
		"name": "completed",
		"type": "bool"
		}
		],
		"name": "TaskCreated",
		"type": "event"
		},
		{
		"inputs": [
		{
		"internalType": "string",
		"name": "_content",
		"type": "string"
		}
		],
		"name": "createTask",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
		},
		{
		"inputs": [],
		"name": "taskCount",
		"outputs": [
		{
		"internalType": "uint256",
		"name": "",
		"type": "uint256"
		}
		],
		"stateMutability": "view",
		"type": "function"
		},
		{
		"inputs": [
		{
		"internalType": "uint256",
		"name": "",
		"type": "uint256"
		}
		],
		"name": "tasks",
		"outputs": [
		{
		"internalType": "uint256",
		"name": "id",
		"type": "uint256"
		},
		{
		"internalType": "string",
		"name": "content",
		"type": "string"
		},
		{
		"internalType": "bool",
		"name": "completed",
		"type": "bool"
		}
		],
		"stateMutability": "view",
		"type": "function"
		},
		{
		"inputs": [
		{
		"internalType": "uint256",
		"name": "_id",
		"type": "uint256"
		}
		],
		"name": "toggleCompleted",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
		}
		], '0x19Db4baa9d9a41C8577dbE89b3C1400541422d52');
}

async  function  getCurrentAccount() {
	const  accounts  =  await  window.web3.eth.getAccounts();
	return  accounts[0];
}

async  function  render() {
	// Prevent double render
	if (window.loading) {
			return
	}

	// Update app loading state
	setLoading(true);

	// Render Account
	$('#account').html(window.account);

	// Render Tasks
	await  window.renderTasks();

	// Update loading state
	setLoading(false);
}

async  function  renderTasks() {
	// Load the total task count from the blockchain
	const  taskCount  =  await  window.contract.methods.taskCount().call();
	const  $taskTemplate  =  $('.taskTemplate');
	  

	// Render out each task with a new task template
	for (var  i  =  1; i  <=  taskCount; i++) {

		// Fetch the task data from the blockchain
		const  task  =  await  window.contract.methods.tasks(i).call();
		const  taskId  =  task[0];
		const  taskContent  =  task[1];
		const  taskCompleted  =  task[2];

		// Create the html for the task
		const  $newTaskTemplate  =  $taskTemplate.clone();
		$newTaskTemplate.find('.content').html(taskContent);
		$newTaskTemplate.find('input')
			.prop('name', taskId)
			.prop('checked', taskCompleted)
			.on('click', toggleCompleted);
	  
	// Put the task in the correct list
	if (taskCompleted) {
			$('#completedTaskList').append($newTaskTemplate);
	} else {
		$('#taskList').append($newTaskTemplate);
	}
	
	// Show the task
		$newTaskTemplate.show();
	}
}

async  function  createTask() {
	setLoading(true);
	const  content  =  $('#newTask').val();
	await  window.contract.methods.createTask(content).send({ from: window.account });
	window.location.reload();
}

async  function  toggleCompleted(e) {
	setLoading(true);
	const  taskId  =  e.target.name;
	await  window.contract.methods.toggleCompleted(taskId).send({ from: window.account });
	window.location.reload();
}

function  setLoading(boolean) {
	window.loading  =  boolean;
	const  loader  =  $('#loader');
	const  content  =  $('#content'); 

	if (boolean) {
		loader.show();
		content.hide();
	} else {
		loader.hide();
		content.show();
	}
}

load();
 ```
### Exécution du code 

```console
$ node server.js 
```

### Code sur github 
