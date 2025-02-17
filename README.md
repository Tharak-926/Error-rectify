const express = require('express')
const {open} = require('sqlite')
const app = express()
app.use(express.json())
const path = require('path')
const dbpath = path.join(__dirname, 'cricketTeam.db')
const sqlite3 = require('sqlite3')

let db = null
const initializeDBAndServer = async () => {
  try {
    db = await open({
      filename: dbpath,
      driver: sqlite3.Database,
    })
    app.listen(3000, () => {
      console.log('Server is Running at http://localhost:3000/')
    })
  } catch (e) {
    console.log(`DBError:${e.message}`)
    process.exit(1)
  }
}
initializeDBAndServer()

const responseObject = dbObject => {
  return {
    playerId: dbObject.player_id,
    playerName: dbObject.player_name,
    jerseyNumber: dbObject.jersey_number,
    role: dbObject.role,
  }
}

app.get('/players/', async (request, response) => {
  const getPlayerQuery = `
  SELECT *
  FROM cricket_team;`
  const player = await db.all(getPlayerQuery)
  response.send(player.map(i => responseObject(i)))
})

app.post('/palyers/', async (request, response) => {
  const playerDetails = request.body
  const {playerName, jerseyNumber, role} = playerDetails
  const post_api = `
  INSERT INTO
  cricket_team (player_name,jersey_number,role)
  VALUES(
    "${playerName}",
    ${jerseyNumber},
    "${role}";
  )`
  const post_db = await db.run(post_api)
  response.send('Player Added to Team')
})

app.get('/players/:playerId/', async (request, response) => {
  const {playerId} = request.params
  const get_api = `
  SELECT *
  FROM cricket_team
  WHERE player_id = ${playerId};`
  const get_db = await db.get(get_api)
  response.send(responseObject(get_db))
})

app.put('/players/:playerId/', async (request, response) => {
  const {playerId} = request.params
  const playerDetails = request.body
  const {playerName, jerseyNUmber, role} = playerDetails
  const put_api = `
  UPDATE cricket_team
  SET 
  player_name = "${playerName}",
  jersey_number = ${jerseyNUmber},
  role = "${role}"
  WHERE player_id = ${playerId};`
  const put_db = await db.run(put_api)
  response.send('Player Details Updated')
})

app.delete('/players/:playerId/', async (request, response) => {
  const {playerId} = request.params
  const delete_api = `
  DELETE 
  FROM cricket_team
  WHERE player_id = ${playerId};`
  const delete_db = await db.run(delete_api)
  response.send('Player Removed')
})
module.exports = app
