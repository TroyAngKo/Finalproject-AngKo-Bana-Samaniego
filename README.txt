READ ME
*Note: This project is created directly using mongoDB version 3.4 which can be downloaded from their site: https://www.mongodb.com/ 

1.) Retrieving the dataset
	- clone or download the github repository from https://github.com/TroyAngKo/Finalproject-AngKo-Bana-Samaniego	
	- The edited dataset (taken from https://www.kaggle.com/imcoates/nba-stats/data) should be included inside the git repository named "Seasons_Stats.csv"
	- For the edited dataset, blank rows that act as space have been removed
2.) Loading the dataset
	- Loading the dataset can be done manually by simply copying and pasting the data on the command line after logging in to the server OR using mongoimport function found in this guide: 		https://docs.mongodb.com/manual/reference/program/mongoimport/
2.) Replication
	- Create a new folder and place the dataset file inside
	- Open powershell and place go to the directory of the folder
	- create two directories  "mongo1" and "mongo2" 
	- To start a server on a new port (27010),run the following code: 
		& 'C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe' --replSet book --rest --httpinterface --port 27010 --dbpath mongo1
	- Open another powershell, go to the folder directory, and create another port (27011) using the code:
		& 'C:\Program Files\MongoDB\Server\3.2\bin\mongod.exe' --replSet book --rest --httpinterface --port 27011 --dbpath mongo2
	- Log-in to the mongo server port 27010 and run rs.initiate()
		var cfg = {
		"_id": "book",
		"version": 1,
		"members": [
			{
			"_id": 0,
			"host": "localhost:27010",
			"priority": 1
			},
				{
			"_id": 1,
			"host": "localhost:27011",
			"priority": 0
				}
			]
		}
			
	- Use rs.status() on the main port till status becomes the primary
	- Use rs.slaveOk() for the replicates

3.) MapReduce
	- The MapReduce function checks the total number of points per game against age, then removes any duplicate/redundant name in the dataset	
	- To execute the MapReduce function, use the following block of code:
		map = function() {
			emit(this.Age, {
				numberOfGames: this.G,
				totalPoints: this.PTS
			})
		}

		reduce = function (key, values) {
		var totalPts = 0, totalGames = 0;
		values.forEach( function(item) {
		if (isNaN(item.totalPoints) == False && isNaN(item.numberOfGames) == False) {
			totalPts += item.totalPoints;
			totalGames += item.numberOfGames;
			}
		});
			return totalPts / totalGames;
		}
		
		reduce = function(key, values) {
			for (var i = values.length; i--;) {
				count += values[i];
				}
				return count;
			}

		results = db.runCommand({
			mapReduce: 'nba',
			map: map,
			reduce: reduce,
			out: 'players.answer1'
			})

		db.players.answer1.find().pretty();
4.) Sharding
	- Use the code --shardsvr to stop the servers
	- Run config server
	- Open another powershell while using a new port (27012) for the config server and run rs.initiate()
		var cfg = {
		"_id": "book",
		"version": 1,
		"configsvr": true,
		"members": [
			{
			"_id": 0,
			"host": "localhost:27012",
			"priority": 1
				}
			]
		}
	- Open another powershell and a setup a new port (20713) as the router server, connect to the config server, and then log-in to the router server
	- Use sh.addShard to add the servers to the collection

 
		
				






