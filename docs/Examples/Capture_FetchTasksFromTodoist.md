### Capture: Fetch Tasks From Todoist
For this capture to work, you will need the Todoist plugin for Obsidian.
You will also need to set it up with your API key.

This is very useful for capturing tasks on the go with your phone, and then adding them to Obsidian when you get back to your computer.

You will need to set up a [macro](../Choices/MacroChoice.md) with the following script.

**IMPORTANT:** If you do _NOT_ want this script to complete tasks in Todoist that you put into your vault, remove the function call to ``closeSelectedTasks`` by removing the line `await closeSelectedTasks(params.app, selectedTasks);` in `start`.

````js
module.exports = start;

const getTodoistPluginApi = (app) => app.plugins.plugins["todoist-sync-plugin"].api;

async function getTasks(params) {
    const api = getTodoistPluginApi(params.app);
    const {ok: tasks} = await api.getTasks();
    return tasks;
}

async function selectTasks(params) {
    const tasks = await getTasks(params);
    const selectedTaskNames = await params.quickAddApi.checkboxPrompt(tasks.map(task => task.content));
    return tasks.filter(task => selectedTaskNames.some(t => t.contains(task.content)));
}

async function closeSelectedTasks(app, selectedTasks) {
    const api = getTodoistPluginApi(app);
    selectedTasks.forEach(async task => await api.closeTask(task.id));
}

function formatTasksToTasksPluginTask(selectedTasks) {
    return selectedTasks.map(task => `- [ ] ${task.content} 📅 ${task.rawDatetime.format("YYYY-MM-DD")}`).join("\n");
}

async function start(params) {
    const selectedTasks = await selectTasks(params);
    await closeSelectedTasks(params.app, selectedTasks);
    return formatTasksToTasksPluginTask(selectedTasks);
}
````

Now, you will need a [Capture choice](docs/Choices/CaptureChoice.md) with the following settings.

- _Capture To File Name:_ the path to the file where you want to store the tasks.
- _Capture format:_ Enabled - and in the format, write``{{MACRO:<MACRONAME>}}`` where `MACRONAME` is the name of the macro that you made earlier.

The tasks are written in this format:
``- [ ] <Task Content> 📆 <DD-MM-YYYY>``

Which equals: ``- [ ] Buy groceries 📆 25/06/2021``

This task will be recognized by the Tasks plugin for Obsidian, as well.

### Steps
1. Set up the Todoist plugin - grab the API key from your Todoist account. There's a link in the plugin's settings.
2. Grab the code block from the example and add it to your vault as a javascript file. I'd encourage you to call it something like todoistTaskSync.js to be explicit.
3. Follow along with what I do in the gif below

![GKkCNWZHLv](https://user-images.githubusercontent.com/29108628/123500983-26ad2880-d642-11eb-9e45-b537271312d1.gif)

### Installation video
https://user-images.githubusercontent.com/29108628/123511101-bde4a100-d67f-11eb-90c1-5bd146c5d0f2.mp4
