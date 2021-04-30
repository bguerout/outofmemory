# dashrun

Run a node.js script and watch process execution on a terminal dashboard.

```shell
npx dashrun your_script.js [arg...]
```

dashrun comes with a built-in memory dashboard and allows you to create custom ones.

![image](https://user-images.githubusercontent.com/221211/113982536-2d00e500-9849-11eb-83b9-7bf2c5fdee0a.png)

## Create a custom dashboard

### Probe

You need first to create a `probe` to send message from the forked process to the dashboard.

A probe can be a simple line of code in your script:

```js
let message = { type: "memory", data: process.memoryUsage() };
process.send(message);
```

or a file:

```js
//myProbe.js (dashrun will automatically inject this file into your script)
setInterval(() => {
  //Sending every second to the dashboard a message with the memory usage of the forked script
  process.send({ type: "memory", data: process.memoryUsage() });
}, 1000);
```

Note that message must be sent
with [process.send](https://nodejs.org/api/process.html#process_process_send_message_sendhandle_options_callback)
core function.

### Dashboard

You are free to use any library to create a dashboard.

[blessed-contrib](https://www.npmjs.com/package/blessed-contrib) is a good choice and dashrun comes with a thin layer
over it to ease dashboard construction.

```js
const path = require("path");
const contrib = require("blessed-contrib");
const { DashboardLayout } = require("dashrun");

//A dashboard is just a function taking a callback to run the script
module.exports = (run) => {

  //Create a layout and define area where bless-contrib components will be rendered
  let layout = new DashboardLayout();
  let top = layout.area(0, 0, 6, 12);

  //Create a component
  let component = contrib.lcd({
    label: "Current value",
    color: "red",
    ...top, //Set area
  });
  layout.add(component);
  layout.render();

  //Run the script
  let probe = path.join(__dirname, "myProbe.js");
  let script = run([probe]);

  //Listen to events sent by the probe and update component
  script.on("memory", (usage) => {
    component.setDisplay(usage.rss);
    layout.render();
  });
};
```
