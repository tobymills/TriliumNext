// Trilium Widget Script: Open Tasks
// Relevant Documentation:
// https://triliumnext.github.io/Docs/Wiki/frontend-basics.html
// https://triliumnext.github.io/Docs/Wiki/widget-basics.html
// https://triliumnext.github.io/Notes/frontend_api/BasicWidget.html

// Widget template definition
const template = `
  <div id="my-widget" style="padding: 10px; border-top: 1px solid var(--main-border-color); contain: none;">
    <h2>Open Tasks</h2>
    <span id="todo-list" class="todo-list__label"></span>
  </div>
`;

// Get widget position from note label or default to "bottom"
const position = api.startNote.getLabelValue("todolistposition") ?? "bottom";

// Widget class definition
class ToDoListWidget extends api.NoteContextAwareWidget {
    // Define the parent widget pane based on position
    get parentWidget() { return ToDoListWidget.parentWidget; }
    uncheckedTasks = []; // Array to hold unchecked tasks

    // Determine widget position
    get position() { return position === "bottom" ? 100 : position === "right" ? 1 : 50; }
    static get parentWidget() { return (position === "bottom" || position === "right") ? "center-pane" : "right-pane"; }

    // Enable the widget for specific note types and labels
    isEnabled() {
        return super.isEnabled() && this.note.type === 'text' && (this.note.hasLabel('todo') || this.note.hasLabel('todolist'));
    }

    // Initialize widget-specific CSS
    init() {
        this.cssBlock(`
          #my-widget {
            overflow: scroll;
            height: 25%;
            z-index: 1;
          }
        `);
    }

    // Main render method
    doRender() {
        this.$widget = $(template); // Load the widget template
        this.$todolist = this.$widget.find('#todo-list'); // Task list container
        this.init();
        this.getContent();
        this.tasksRender(); // Render tasks
        return this.$widget;
    }

    // Alternative render method for the widget body
    doRenderBody() {
        this.$template = $(template);
        this.$todolist = this.$template.find('#todo-list');
        this.init();
        this.getContent();
        this.tasksRender();
        return this.$template;
    }

    // Fetch content and extract unchecked tasks
    async getContent() {
        this.uncheckedTasks = []; // Reset tasks
        let searchStr = `note.ancestors.noteId='${this.noteId}' and (~template='Day template' or #todo)`;
        let childNotes = await api.searchForNotes(searchStr);

        for (let note of childNotes) {
            const { content } = await note.getBlob(); // Get note content
            let taskRegex = /<label class="todo-list__label">\s*<input type="checkbox"[^>]*>\s*<span class="todo-list__label__description">([^<]+)<\/span>\s*<\/label>\s*/g;
            if (content) {
                let match;
                let tempTask;
                let counter = 0;

                // Extract unchecked tasks using regex
                while ((match = taskRegex.exec(content)) !== null) {
                    if (match[1] != "&nbsp;" && match[0].indexOf("checked") < 0) {
                        tempTask = new Task(note.noteId, note.title, counter, match[1].trim());
                        this.uncheckedTasks.push(tempTask);
                    }
                    counter++;
                }
            }
        }
    }

    // Refresh tasks when note content changes
    async refreshWithNote(note) {    
        await this.getContent();
        this.tasksRender();
    }

    // Handle entity reload events
    async entitiesReloadedEvent({ loadResults }) {
        if (loadResults.isNoteContentReloaded(this.noteId)) {
            this.refresh();
        }
    }

    // Update task checkbox status
    static async updateTask(noteId, taskIndex, checked) {
        let data = await api.runOnBackend((noteId) => {
            const note = api.searchForNote(`note.noteId="${noteId}"`);
            return note ? note.getContent() : null;
        }, [noteId]);

        let $tmpnote = $(data);
        $tmpnote.find("input[type='checkbox']").eq(taskIndex).attr("checked", checked);

        let content = $tmpnote.map((index, elem) => $(elem).prop("outerHTML")).toArray();

        await api.runOnBackend((noteId, content) => {
            const note = api.getNote(noteId);
            if (note) {
                note.setContent(content);
            } else {
                api.log("Note not found: " + noteId);
            }
        }, [noteId, content]);
    }

    // Render tasks into the widget
    tasksRender() {
        this.$todolist.empty(); // Clear current tasks
        let currentNoteTitle = ''; // Track the current note to group tasks

        // Render each task
        this.uncheckedTasks.forEach(task => {
            if (currentNoteTitle !== task.noteId) {
                // Add the note title if it's a new group
                this.$todolist.append(
                  `<h3 class="task"><a href="/root/${task.noteId}" id="${task.noteId}" class="noteLinks">${task.noteTitle}</a></h3>`
                );
                currentNoteTitle = task.noteId;
            }
            // Add the task itself
            this.$todolist.append(`<div class="task">${task.displayTask()}</div>`);
        });

        // Attach events for task links and checkboxes
        this.$widget.find(".noteLinks").on("click", function () {
            api.openTabWithNote($(this).attr("id"), true);
        });

        this.$widget.find("input:checkbox").on("click", async function () {
            const noteId = $(this).attr("data-noteId");
            const taskId = $(this).attr("data-taskId");
            const checked = $(this).prop("checked");
            await ToDoListWidget.updateTask(noteId, taskId, checked);
        });
    }
}

// Task class to represent individual tasks
class Task {
    constructor(noteId, noteTitle, taskIndex, taskText) {
        this.noteId = noteId;       // Note ID
        this.noteTitle = noteTitle; // Note title
        this.taskIndex = taskIndex; // Task index in the note
        this.taskText = taskText;   // Task description
    }

    // Render task details
    displayTask() {
        return `
          <span contenteditable="false">
            <input type="checkbox" tabindex="-1" data-noteId="${this.noteId}" data-taskId="${this.taskIndex}">
          </span>
          <span class="todo-list__label__description">${this.taskText}</span>
        `;
    }
}

// Export the widget
module.exports = new ToDoListWidget();
