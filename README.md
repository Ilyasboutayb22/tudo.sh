Le code:
#!/bin/bash

# File to store tasks
TASK_FILE="tasks.csv"

# Initialize the task file if it doesn't exist
if [[ ! -f "$TASK_FILE" ]]; then
    echo "id,title,description,location,due_date,due_time,completed" > "$TASK_FILE"
fi

# Function to generate a unique identifier
generate_id() {
    echo $(( $(tail -n 1 "$TASK_FILE" | cut -d',' -f1) + 1 ))
}

# Function to validate date
validate_date() {
    date -d "$1" "+%Y-%m-%d" >/dev/null 2>&1
}

# Function to validate time
validate_time() {
    date -d "$1" "+%H:%M" >/dev/null 2>&1
}

# Function to prompt user for task details
get_task_details() {
    read -p "Title (required): " title
    if [[ -z "$title" ]]; then
        echo "Title is required." >&2
        exit 1
    fi
    read -p "Description (optional): " description
    read -p "Location (optional): " location
    read -p "Due Date (YYYY-MM-DD, required): " due_date
    if ! validate_date "$due_date"; then
        echo "Invalid due date." >&2
        exit 1
    fi
    read -p "Due Time (HH:MM, required): " due_time
    if ! validate_time "$due_time"; then
        echo "Invalid due time." >&2
        exit 1
    fi
    completed="no"
}

# Function to create a task
create_task() {
    get_task_details
    id=$(generate_id)
    echo "$id,$title,$description,$location,$due_date,$due_time,$completed" >> "$TASK_FILE"
    echo "Task created successfully."
}

# Function to update a task
update_task() {
    read -p "Enter task ID to update: " id
    if ! grep -q "^$id," "$TASK_FILE"; then
        echo "Task with ID $id not found." >&2
        exit 1
    fi
    get_task_details
    sed -i "/^$id,/c\\$id,$title,$description,$location,$due_date,$due_time,$completed" "$TASK_FILE"
    echo "Task updated successfully."
}

# Function to delete a task
delete_task() {
    read -p "Enter task ID to delete: " id
    if ! grep -q "^$id," "$TASK_FILE"; then
        echo "Task with ID $id not found." >&2
        exit 1
    fi
    sed -i "/^$id,/d" "$TASK_FILE"
    echo "Task deleted successfully."
}

# Function to show all information about a task
show_task() {
    read -p "Enter task ID to view: " id
    if ! grep -q "^$id," "$TASK_FILE"; then
        echo "Task with ID $id not found." >&2
        exit 1
    fi
    grep "^$id," "$TASK_FILE" | awk -F, '{print "ID: "$1"\nTitle: "$2"\nDescription: "$3"\nLocation: "$4"\nDue Date: "$5"\nDue Time: "$6"\nCompleted: "$7}'
}

# Function to list tasks of a given day
list_tasks() {
    read -p "Enter date (YYYY-MM-DD): " date
    if ! validate_date "$date"; then
        echo "Invalid date." >&2
        exit 1
    fi
    echo "Completed tasks:"
    awk -F, -v date="$date" '$5 == date && $7 == "yes" {print $0}' "$TASK_FILE"
    echo "Uncompleted tasks:"
    awk -F, -v date="$date" '$5 == date && $7 == "no" {print $0}' "$TASK_FILE"
}

# Function to search for a task by title
search_task() {
    read -p "Enter title to search: " title
    grep -i ",$title," "$TASK_FILE" || echo "No tasks found with title '$title'."
}

# Function to display help menu
display_help() {
    echo "Usage: $0 {create|update|delete|show|list|search|help}"
    echo
    echo "Options:"
    echo "  create           Create a new task"
    echo "  update           Update an existing task"
    echo "  delete           Delete a task"
    echo "  show             Show details of a task"
    echo "  list             List tasks for a specific date"
    echo "  search           Search tasks by title"
    echo "  help             Display this help menu"
    echo
    echo "When run without arguments, it displays completed and uncompleted tasks for the current day."
}

# Display today's tasks if no arguments are provided
if [[ $# -eq 0 ]]; then
    today=$(date "+%Y-%m-%d")
    echo "Completed tasks for $today:"
    awk -F, -v date="$today" '$5 == date && $7 == "yes" {print $0}' "$TASK_FILE"
    echo "Uncompleted tasks for $today:"
    awk -F, -v date="$today" '$5 == date && $7 == "no" {print $0}' "$TASK_FILE"
    exit 0
fi

# Parse command-line arguments
case "$1" in
    create)
        create_task
        ;;
    update)
        update_task
        ;;
    delete)
        delete_task
        ;;
    show)
        show_task
        ;;
    list)
        list_tasks
        ;;
    search)
        search_task
        ;;
    help)
        display_help
        ;;
    *)
        echo "Invalid option. Use '$0 help' to see the valid options." >&2
        exit 1
        ;;
esac

desciption:
markdown
Copier le code
# Gestionnaire de Tâches

Ceci est un script Bash simple pour gérer des tâches. Les tâches sont stockées dans un fichier CSV et peuvent être créées, mises à jour, supprimées, affichées, listées et recherchées à l'aide de ce script.

## Utilisation


./task_manager.sh {create|update|delete|show|list|search|help}
Commandes
create
Créer une nouvelle tâche.


./task_manager.sh create
update
Mettre à jour une tâche existante par son ID.


./task_manager.sh update
delete
Supprimer une tâche par son ID.


./task_manager.sh delete
show
Afficher les détails d'une tâche par son ID.



./task_manager.sh show
list
Lister les tâches pour une date spécifique (YYYY-MM-DD).



./task_manager.sh list
search
Rechercher des tâches par titre.



./task_manager.sh search
help
Afficher le menu d'aide.



./task_manager.sh help
Fonctionnalités
Les tâches sont stockées dans un fichier CSV (tasks.csv).
Chaque tâche comporte les champs suivants :
ID
Titre
Description
Lieu
Date d'échéance
Heure d'échéance
Complétée
Le script valide les dates et les heures.
Lorsqu'il est exécuté sans arguments, il affiche les tâches complétées et non complétées pour la journée en cours.
Structure du Fichier
tasks.csv : Le fichier où les tâches sont stockées. Il est créé automatiquement s'il n'existe pas.
Fonctions
generate_id
Génère un identifiant unique pour chaque tâche.

validate_date
Valide le format de la date (YYYY-MM-DD).

validate_time
Valide le format de l'heure (HH
).

get_task_details
Demande à l'utilisateur de saisir les détails de la tâche.

create_task
Crée une nouvelle tâche et l'ajoute au fichier CSV.

update_task
Met à jour une tâche existante dans le fichier CSV.

delete_task
Supprime une tâche du fichier CSV.

show_task
Affiche les détails d'une tâche.

list_tasks
Liste toutes les tâches pour une date spécifique, catégorisées en complétées et non complétées.

search_task
Recherche des tâches par titre.

display_help
Affiche le menu d'aide.

Exemple
Création d'une nouvelle tâche :


./task_manager.sh create
# Suivez les instructions pour entrer les détails de la tâche
Mise à jour d'une tâche :



./task_manager.sh update
# Suivez les instructions pour mettre à jour les détails de la tâche
Suppression d'une tâche :



./task_manager.sh delete
# Suivez les instructions pour entrer l'ID de la tâche à supprimer
Liste des tâches pour une date spécifique :

bash

./task_manager.sh list
# Suivez les instructions pour entrer la date
Recherche de tâches par titre :



./task_manager.sh search
# Suivez les instructions pour entrer le titre à rechercher
Affichage des détails d'une tâche :


./task_manager.sh show
# Suivez les instructions pour entrer l'ID de la tâche
Affichage de l'aide :



./task_manager.sh help
Lorsqu'il est exécuté sans arguments, le script affiche les tâches complétées et non complétées pour la journée en cours.

vbnet
