# Ex-Ray - a cloud-native debugger for production Golang systems
###### Ask your production software anything


![Ex-Ray iceberg](https://github.com/DataExMachina-dev/exray-public/blob/main/iceberg.png)


Ex-Ray is a new-generation debugger for complex Go services. In the shift towards cloud computing and micro-services, we have lost an essential development and support tool – the debugger. Debuggers are great for desktop software but don’t quite apply to cloud software (your programs run on many machines, not just one; processes cannot be “paused” for any measurable amount of time and certainly a human cannot be in the loop while a process is stopped, no single person doesn’t understand all the code of complex systems). At the same time, software has been getting ever more complex; now we need advanced observability tooling more than ever. We don’t have to throw out the baby with the bathwater. Ex-Ray, from Data Ex Machina, is a next-generation debugger focused on distributed Golang systems running in production. It aims to make debugging pleasant and exciting by bringing back the ability to “ask your software anything”.

#### Features
- Collect “snapshots” of a whole distributed system
- Explore the execution state of all goroutines at a certain point in time
- Collect interesting data from stacks and heap
- Produce reports customized to your programs
- Query and explore the collected data with SQL
- Present the data in Grafana dashboards



