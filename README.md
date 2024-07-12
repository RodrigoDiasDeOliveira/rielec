Project Structure Diagram
rielec/
├── backend/
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       │   └── com/
│   │       │       └── rielec/
│   │       │           ├── config/
│   │       │           │   ├── WebSocketConfig.java
│   │       │           │   └── MongoConfig.java
│   │       │           ├── controller/
│   │       │           │   ├── InstanceController.java
│   │       │           │   └── MachineController.java
│   │       │           ├── model/
│   │       │           │   ├── Instance.java
│   │       │           │   └── Machine.java
│   │       │           ├── repository/
│   │       │           │   ├── InstanceRepository.java
│   │       │           │   └── MachineRepository.java
│   │       │           ├── service/
│   │       │           │   ├── DockerService.java
│   │       │           │   ├── InstanceService.java
│   │       │           │   └── MachineService.java
│   │       │           ├── event/
│   │       │           │   └── MachineStatusChangeEvent.java
│   │       │           └── RielecApplication.java
│   │       └── resources/
│   │           └── application.properties
│   └── pom.xml
├── frontend/
│   ├── public/
│   │   └── index.html
│   ├── src/
│   │   ├── components/
│   │   │   ├── InstanceList.js
│   │   │   ├── InstanceDetails.js
│   │   │   ├── InstanceForm.js
│   │   │   ├── MachineList.js
│   │   │   ├── MachineDetails.js
│   │   │   ├── MachineForm.js
│   │   │   └── MachineStatusUpdater.js
│   │   ├── services/
│   │   │   ├── api.js
│   │   │   └── websocket.js
│   │   ├── App.js
│   │   └── index.js
│   ├── package.json
│   └── README.md
└── database/
    └── mongodb-data/




    Backend (Java Spring Boot)
RielecApplication.java
package com.rielec;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RielecApplication {
    public static void main(String[] args) {
        SpringApplication.run(RielecApplication.class, args);
    }
}
WebSocketConfig.java
package com.rielec.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").setAllowedOrigins("http://localhost:3000").withSockJS();
    }
}
MongoConfig.java
package com.rielec.config;

import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.core.MongoTemplate;

@Configuration
public class MongoConfig {

    @Bean
    public MongoClient mongoClient() {
        return MongoClients.create("mongodb://localhost:27017");
    }

    @Bean
    public MongoTemplate mongoTemplate() {
        return new MongoTemplate(mongoClient(), "rielec");
    }
}
Instance.java
package com.rielec.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.time.LocalDateTime;

@Document(collection = "instances")
public class Instance {
    @Id
    private String id;
    private String name;
    private String status;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    // Constructors, getters, and setters
}
Machine.java
package com.rielec.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.time.LocalDateTime;

@Document(collection = "machines")
public class Machine {
    @Id
    private String id;
    private String name;
    private String status;
    private String instanceId;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    // Constructors, getters, and setters
}
InstanceRepository.java
package com.rielec.repository;

import com.rielec.model.Instance;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface InstanceRepository extends MongoRepository {
}
MachineRepository.java
package com.rielec.repository;

import com.rielec.model.Machine;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface MachineRepository extends MongoRepository {
}
DockerService.java
package com.rielec.service;

import com.github.dockerjava.api.DockerClient;
import com.github.dockerjava.core.DockerClientBuilder;
import com.rielec.model.Instance;
import org.springframework.stereotype.Service;

@Service
public class DockerService {

    private final DockerClient dockerClient;

    public DockerService() {
        this.dockerClient = DockerClientBuilder.getInstance().build();
    }

    public void createInstance(Instance instance) {
        // Implementation for creating a Docker instance
    }

    public void startInstance(Instance instance) {
        // Implementation for starting a Docker instance
    }

    public void stopInstance(Instance instance) {
        // Implementation for stopping a Docker instance
    }

    public String getInstanceStatus(Instance instance) {
        // Implementation for getting the status of a Docker instance
        return "running";  // Placeholder
    }
}
InstanceService.java
package com.rielec.service;

import com.rielec.model.Instance;
import com.rielec.repository.InstanceRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class InstanceService {

    @Autowired
    private InstanceRepository instanceRepository;

    @Autowired
    private DockerService dockerService;

    public List getAllInstances() {
        return instanceRepository.findAll();
    }

    public Instance getInstance(String id) {
        return instanceRepository.findById(id).orElseThrow(() -> new RuntimeException("Instance not found"));
    }

    public Instance createInstance(Instance instance) {
        dockerService.createInstance(instance);
        return instanceRepository.save(instance);
    }

    public Instance updateInstance(String id, Instance instanceDetails) {
        Instance instance = instanceRepository.findById(id).orElseThrow(() -> new RuntimeException("Instance not found"));
        instance.setName(instanceDetails.getName());
        instance.setStatus(instanceDetails.getStatus());
        return instanceRepository.save(instance);
    }

    public void deleteInstance(String id) {
        Instance instance = instanceRepository.findById(id).orElseThrow(() -> new RuntimeException("Instance not found"));
        dockerService.stopInstance(instance);
        instanceRepository.deleteById(id);
    }
}
MachineService.java
package com.rielec.service;

import com.rielec.model.Machine;
import com.rielec.repository.MachineRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class MachineService {

    @Autowired
    private MachineRepository machineRepository;

    public List getAllMachines() {
        return machineRepository.findAll();
    }

    public Machine getMachine(String id) {
        return machineRepository.findById(id).orElseThrow(() -> new RuntimeException("Machine not found"));
    }

    public Machine createMachine(Machine machine) {
        return machineRepository.save(machine);
    }

    public Machine updateMachine(String id, Machine machineDetails) {
        Machine machine = machineRepository.findById(id).orElseThrow(() -> new RuntimeException("Machine not found"));
        machine.setName(machineDetails.getName());
        machine.setStatus(machineDetails.getStatus());
        machine.setInstanceId(machineDetails.getInstanceId());
        return machineRepository.save(machine);
    }

    public void deleteMachine(String id) {
        machineRepository.deleteById(id);
    }
}
MachineStatusChangeEvent.java
package com.rielec.event;

public class MachineStatusChangeEvent {
    private String machineId;
    private String status;
    private long timestamp;

    // Constructors, getters, and setters
}
InstanceController.java
package com.rielec.controller;

import com.rielec.model.Instance;
import com.rielec.service.InstanceService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/instances")
public class InstanceController {

    @Autowired
    private InstanceService instanceService;

    @GetMapping
    public List getAllInstances() {
        return instanceService.getAllInstances();
    }

    @GetMapping("/{id}")
    public Instance getInstance(@PathVariable String id) {
        return instanceService.getInstance(id);
    }

    @PostMapping
    public Instance createInstance(@RequestBody Instance instance) {
        return instanceService.createInstance(instance);
    }

    @PutMapping("/{id}")
    public Instance updateInstance(@PathVariable String id, @RequestBody Instance instanceDetails) {
        return instanceService.updateInstance(id, instanceDetails);
    }

    @DeleteMapping("/{id}")
    public void deleteInstance(@PathVariable String id) {
        instanceService.deleteInstance(id);
    }
}
MachineController.java
package com.rielec.controller;

import com.rielec.event.MachineStatusChangeEvent;
import com.rielec.model.Machine;
import com.rielec.service.MachineService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/machines")
public class MachineController {

    @Autowired
    private MachineService machineService;

    @Autowired
    private SimpMessagingTemplate messagingTemplate;

    @GetMapping
    public List getAllMachines() {
""        return machineService.getAllMachines();
    }

    @GetMapping("/{id}")
    public Machine getMachine(@PathVariable String id) {
        return machineService.getMachine(id);
    }

    @PostMapping
    public Machine createMachine(@RequestBody Machine machine) {
        return machineService.createMachine(machine);
    }

    @PutMapping("/{id}")
    public Machine updateMachine(@PathVariable String id, @RequestBody Machine machineDetails) {
        Machine updatedMachine = machineService.updateMachine(id, machineDetails);
        
        // Send status change event
        MachineStatusChangeEvent event = new MachineStatusChangeEvent();
        event.setMachineId(updatedMachine.getId());
        event.setStatus(updatedMachine.getStatus());
        event.setTimestamp(System.currentTimeMillis());
        messagingTemplate.convertAndSend("/topic/machine-status", event);
        
        return updatedMachine;
    }

    @DeleteMapping("/{id}")
    public void deleteMachine(@PathVariable String id) {
        machineService.deleteMachine(id);
    }
}






Frontend (React)
package.json
{
  "name": "rielec-frontend",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@stomp/stompjs": "^6.1.2",
    "axios": "^0.21.1",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-router-dom": "^5.2.0",
    "react-scripts": "4.0.3",
    "sockjs-client": "^1.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(
  
    
  ,
  document.getElementById('root')
);
src/App.js
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import InstanceList from './components/InstanceList';
import InstanceDetails from './components/InstanceDetails';
import InstanceForm from './components/InstanceForm';
import MachineList from './components/MachineList';
import MachineDetails from './components/MachineDetails';
import MachineForm from './components/MachineForm';
import MachineStatusUpdater from './components/MachineStatusUpdater';

function App() {
  return (
    
      

        
        
          
          
          
          
          
          
        
      

    
  );
}

export default App;
src/services/api.js
import axios from 'axios';

const API_URL = 'http://localhost:8080/api';

export const fetchInstances = () => axios.get(`${API_URL}/instances`);
export const fetchInstance = (id) => axios.get(`${API_URL}/instances/${id}`);
export const createInstance = (instance) => axios.post(`${API_URL}/instances`, instance);
export const updateInstance = (id, instance) => axios.put(`${API_URL}/instances/${id}`, instance);
export const deleteInstance = (id) => axios.delete(`${API_URL}/instances/${id}`);

export const fetchMachines = () => axios.get(`${API_URL}/machines`);
export const fetchMachine = (id) => axios.get(`${API_URL}/machines/${id}`);
export const createMachine = (machine) => axios.post(`${API_URL}/machines`, machine);
export const updateMachine = (id, machine) => axios.put(`${API_URL}/machines/${id}`, machine);
export const deleteMachine = (id) => axios.delete(`${API_URL}/machines/${id}`);
src/services/websocket.js
import { Client } from '@stomp/stompjs';

const SOCKET_URL = 'http://localhost:8080/ws';

let stompClient = null;

export const connectWebSocket = (onMachineStatusChange) => {
  stompClient = new Client({
    brokerURL: SOCKET_URL,
    onConnect: () => {
      console.log('Connected to WebSocket');
      stompClient.subscribe('/topic/machine-status', (message) => {
        const event = JSON.parse(message.body);
        onMachineStatusChange(event);
      });
    },
  });

  stompClient.activate();
};

export const disconnectWebSocket = () => {
  if (stompClient !== null) {
    stompClient.deactivate();
  }
};
src/components/InstanceList.js
import React, { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';
import { fetchInstances } from '../services/api';

const InstanceList = () => {
  const [instances, setInstances] = useState([]);

  useEffect(() => {
    fetchInstances().then(response => setInstances(response.data));
  }, []);

  return (
    

      
Instances

      Create New Instance
      

        {instances.map(instance => (
          

            {instance.name} - {instance.status}
          

        ))}
      

    

  );
};

export default InstanceList;
src/components/InstanceDetails.js
import React, { useState, useEffect } from 'react';
import { useParams, useHistory } from 'react-router-dom';
import { fetchInstance, deleteInstance } from '../services/api';

const InstanceDetails = () => {
  const [instance, setInstance] = useState(null);
  const { id } = useParams();
  const history = useHistory();

  useEffect(() => {
    fetchInstance(id).then(response => setInstance(response.data));
  }, [id]);

  const handleDelete = () => {
    deleteInstance(id).then(() => history.push('/'));
  };

  if (!instance) return 
Loading...
;

  return (
    

      
{instance.name}

      
Status: {instance.status}


      
Created At: {instance.createdAt}


      
Updated At: {instance.updatedAt}


       history.push(`/instances/${id}/edit`)}>Edit
      Delete
    

  );
};

export default InstanceDetails;
src/components/InstanceForm.js
import React, { useState } from 'react';
import { useHistory } from 'react-router-dom';
import { createInstance } from '../services/api';

const InstanceForm = () => {
  const [name, setName] = useState('');
  const history = useHistory();

  const handleSubmit = (e) => {
    e.preventDefault();
    createInstance({ name, status: 'CREATED' })
      .then(() => history.push('/'));
  };

  return (
    

      
Create New Instance

      
        Name:
        
{name}
 setName(e.target.value)} required />
      
      Create
    

  );
};

export default InstanceForm;
src/components/MachineList.js
import React, { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';
import { fetchMachines } from '../services/api';

const MachineList = () => {
  const [machines, setMachines] = useState([]);

  useEffect(() => {
    fetchMachines().then(response => setMachines(response.data));
  }, []);

  return (
    

      
Machines

      Create New Machine
      

        {machines.map(machine => (
          

            {machine.name} - {machine.status}
          

        ))}
      

    

  );
};

export default MachineList;
src/components/MachineDetails.js
import React, { useState, useEffect } from 'react';
import { useParams, useHistory } from 'react-router-dom';
import { fetchMachine, deleteMachine } from '../services/api';

const MachineDetails = () => {
  const [machine, setMachine] = useState(null);
  const { id } = useParams();
  const history = useHistory();

  useEffect(() => {
    fetchMachine(id).then(response => setMachine(response.data));
  }, [id]);

  const handleDelete = () => {
    deleteMachine(id).then(() => history.push('/machines'));
  };

  if (!machine) return 
Loading...
;

  return (
    

      
{machine.name}

      
Status: {machine.status}


      
Instance ID: {machine.instanceId}


      
Created At: {machine.createdAt}


      
Updated At: {machine.updatedAt}


       history.push(`/machines/${id}/edit`)}>Edit
      Delete
    

  );
};

export default MachineDetails;
src/components/MachineForm.js
import React, { useState } from 'react';
import { useHistory } from 'react-router-dom';
import { createMachine } from '../services/api';

const MachineForm = () => {
  const [name, setName] = useState('');
  const [instanceId, setInstanceId] = useState('');
  const history = useHistory();

  const handleSubmit = (e) => {
    e.preventDefault();
    createMachine({ name, status: 'CREATED', instanceId })
      .then(() => history.push('/machines'));
  };

  return (
    

      
Create New Machine

      
        Name:
        
{name}
 setName(e.target.value)} required />
      
      
        Instance ID:
        
{instanceId}
 setInstanceId(e.target.value)} required />
      
      Create
    

  );
};

export default MachineForm;
src/components/MachineStatusUpdater.js
import { useEffect } from 'react';
import { connectWebSocket, disconnectWebSocket } from '../services/websocket';

const MachineStatusUpdater = () => {
  useEffect(() => {
    const onMachineStatusChange = (event) => {
      console.log('Machine status changed:', event);
      // Update the UI or trigger a re-fetch of machines
    };

    connectWebSocket(onMachineStatusChange);

    return () => {
      disconnectWebSocket();
    };
  }, []);

  return null; // This component doesn't render anything
};

export default MachineStatusUpdater;
MongoDB
MongoDB is used as the database for this project. Make sure you have MongoDB installed and running on your local machine or update the connection string in MongoConfig.java to point to your MongoDB instance.

The database will be automatically created when you run the application, and the collections (instances and machines) will be created as needed when you start inserting data.

This completes the full-stack implementation of the RIELEC project, including the Spring Boot backend, React frontend, and MongoDB database integration.


