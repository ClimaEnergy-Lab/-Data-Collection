Node-red 날씨 데이터 수집 flow code 입니다.
Node-red로 실시간 데이터 수집을 위한 flow로, 두 군데의 url 수정이 필요합니다.
1. openweather api
   -> httpRequest1노드에서 url 항목 수정: "https://api.openweathermap.org/data/2.5/weather?lat=37.5666791&lon=126.9782914&appid=d132b2e021cbd63d244a66f0a41fa670&units=metric" -> 이 url은 그냥 사용하면 됩니다.
2. Data storage
   -> httpRequest2노드에서 put your end-point(weatherdata storage)에 url 또는 저장소의 주소를 적으면 됩니다. postgrasql을 사용하는 경우, 노드 수정을 통해 json을 DB 저장소에 저장하도록 수정하면 됩니다.


[
    {
        "id": "e974e044cd2f716a",
        "type": "tab",
        "label": "weatherdataflow",
        "disabled": false,
        "info": "",
        "env": []
    },
    {
        "id": "inject1",
        "type": "inject",
        "z": "e974e044cd2f716a",
        "name": "Trigger every 10 minutes",
        "props": [],
        "repeat": "600",
        "crontab": "",
        "once": true,
        "onceDelay": 0.1,
        "topic": "",
        "x": 150,
        "y": 180,
        "wires": [
            [
                "httpRequest1"
            ]
        ]
    },
    {
        "id": "httpRequest1",
        "type": "http request",
        "z": "e974e044cd2f716a",
        "name": "Get Weather Data",
        "method": "GET",
        "ret": "obj",
        "paytoqs": "ignore",
        "url": "https://api.openweathermap.org/data/2.5/weather?lat=37.5666791&lon=126.9782914&appid=d132b2e021cbd63d244a66f0a41fa670&units=metric",
        "tls": "",
        "persist": false,
        "proxy": "",
        "insecureHTTPParser": false,
        "authType": "",
        "senderr": false,
        "headers": [],
        "x": 400,
        "y": 180,
        "wires": [
            [
                "function1"
            ]
        ]
    },
    {
        "id": "function1",
        "type": "function",
        "z": "e974e044cd2f716a",
        "name": "Process and Forward Weather Data",
        "func": "msg.payload = {\n    \"device_id\": \"openweather_data\",\n    \"location\": msg.payload.name,\n    \"weather\": msg.payload.weather[0].main,\n    \"temp\": msg.payload.main.temp,\n    \"feels_like\": msg.payload.main.feels_like,\n    \"temp_min\": msg.payload.main.temp_min,\n    \"temp_max\": msg.payload.main.temp_max,\n    \"pressure\": msg.payload.main.pressure,\n    \"humidity\": msg.payload.main.humidity,\n    \"sea_level\": msg.payload.main.sea_level,\n    \"grnd_level\": msg.payload.main.grnd_level,\n    \"description\": msg.payload.weather[0].description,\n    \"visibility\": msg.payload.visibility,\n    \"wind_speed\": msg.payload.wind.speed,\n    \"wind_deg\": msg.payload.wind.deg,\n    \"wind_gust\": msg.payload.wind.gust,\n    \"clouds\": msg.payload.clouds.all,\n    \"sunrise\": new Date(msg.payload.sys.sunrise * 1000).toISOString(),  // sunrise를 ISO 8601 형식으로 변환\n    \"sunset\": new Date(msg.payload.sys.sunset * 1000).toISOString(),    // sunset을 ISO 8601 형식으로 변환\n    \"Unixtimestamp\": new Date(msg.payload.dt * 1000).toISOString()      // dt를 ISO 8601 형식으로 변환\n};\nreturn msg;\n",
        "outputs": 1,
        "timeout": "",
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 520,
        "y": 320,
        "wires": [
            [
                "debugNode",
                "httpRequest2"
            ]
        ]
    },
    {
        "id": "httpRequest2",
        "type": "http request",
        "z": "e974e044cd2f716a",
        "name": "Send to Cloud",
        "method": "POST",
        "ret": "txt",
        "paytoqs": "ignore",
        "url": "put your end-point(weatherdata storage)",
        "tls": "",
        "persist": false,
        "proxy": "",
        "insecureHTTPParser": false,
        "authType": "",
        "senderr": false,
        "headers": [],
        "x": 680,
        "y": 120,
        "wires": [
            [
                "statusFunction"
            ]
        ]
    },
    {
        "id": "statusFunction",
        "type": "function",
        "z": "e974e044cd2f716a",
        "name": "Check Status and Log",
        "func": "msg.metadata = msg.metadata || {};\n\nif (msg.statusCode >= 200 && msg.statusCode <= 299) {\n    msg.metadata.status = \"success\";\n    msg.metadata.message = \"Message sent successfully.\";\n    node.status({ fill: 'green', shape: 'dot', text: 'Success' });\n} else {\n    msg.metadata.status = \"failed\";\n    msg.metadata.message = \"Message failed with status code: \" + msg.statusCode;\n    node.status({ fill: 'red', shape: 'ring', text: 'Failed' });\n}\nnode.warn(msg.metadata);\nreturn msg;",
        "outputs": 1,
        "timeout": "",
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 900,
        "y": 320,
        "wires": [
            []
        ]
    },
    {
        "id": "debugNode",
        "type": "debug",
        "z": "e974e044cd2f716a",
        "name": "Debug Output",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "targetType": "msg",
        "x": 880,
        "y": 420,
        "wires": []
    }
]
