- Containerization: packaging application (libraries, source, runtime, config) -> atomic and dynamic. -> ensure consistency of application in every environment
- Container vs VM (Virtual Machine)
- Orchestration
	- Docker: 
		- cons: standard platform for build and run container via Docker Engine, Dockerfiles.
		- pros: Docker Compose -> simple tool for local develop environment. NOT manage complex distribution system in production.
	- In Production
		- Self-Healing
		- Scaling
		- Networking
	- K8s: orchestration platform, auto deployment, scale and manage container
		- K8s + Docker
	* Summary: 
		* Containerization pa