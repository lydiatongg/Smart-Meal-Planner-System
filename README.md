/* TMF 1414 ITP Project Year 1 Sem 1 - Smart Meal Planner System (SMPS)
	Group 07:
	LYDIA TONG SU MIN (104816),
	STEPHEN AUGUSTINE ANAK ALIEW (106446),
	VALERIE VOON KAIXIN (106607),
	AARON NGANG THOMAS (103039),
	EUPHRASIA WAJA DUBLIN (104073)
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h> 
#include <time.h>	//Preproccessors

#define MAX_MEALS 100
#define MAX_NAME_LEN 50
#define MAX_CATEGORY_LENGTH 20
#define MAX_CATEGORIES 5
#define DAYS 7

//Define a structure to hold details for a meal database
typedef struct {
    char name[MAX_NAME_LEN];	//Array to store information about a maximum number of meal names, defined by MAX_NAME_LEN
    char category[MAX_NAME_LEN];	//Array to store information about a maximum number of meal categories, defined by MAX_NAME_LEN
    int calories;	//Stores the calories count
    char dietaryType[MAX_NAME_LEN];	//Array to store information about a maximum number of dietary types, defined by MAX_NAME_LEN
    char date[11];	// Stores the date in the format "YYYY/MM/DD", with space for the null character 
} Meal;

Meal mealDatabase[MAX_MEALS];	
int mealCount = 0;
int calculatedCalorieTarget = 0;
int weeklyCategoryCount[4] = {0};
int dailyCalories[DAYS] = {0};	// Track daily calories for 7 days

void clearInputBuffer();	//Function to clear input buffer
void saveToFile();	//Function to save database to file
void loadFromFile();	//Function to load database from file
void addMeal();	//Function to add meals 
void displayMeals();	//Function to display meals
void deleteMeal();	//Function to delete meal
void updateMeal();	//Function to update meal
void calculateCaloricRequirement();	//Function to calculate caloric requirement
void generateMealPlan();	//Function to generate meal plan
void viewMealHistory();	//Function to view meal history
void to_lower_case(char *str);	//Function to convert all uppercase letters in a given string to lowercase letters


// Function definitions
void clearInputBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

void saveToFile() {
	//Open the file for writing
    //File = meals.txt file pointer
    FILE *file = fopen("meals.txt", "w");
    if (!file) {
    	//Exits program if the file cannot be opened
        printf("Error: Unable to open file for saving.\n");
        return;
    }
    //Loop through each meal
    for (int i = 0; i < mealCount; i++) {
        fprintf(file, "%s,%s,%d,%s\n", mealDatabase[i].name, mealDatabase[i].category, mealDatabase[i].calories, mealDatabase[i].dietaryType);
    }
    //Close the file
    fclose(file);
    printf("Data saved successfully.\n");
}

void loadFromFile() {
    FILE *file = fopen("meals.txt", "r");
    if (!file) {
        printf("No existing data found. Starting fresh.\n");
        return;
    }
    mealCount = 0;
    while (fscanf(file, "%49[^,],%49[^,],%d,%49[^\n]\n", mealDatabase[mealCount].name, mealDatabase[mealCount].category, &mealDatabase[mealCount].calories, mealDatabase[mealCount].dietaryType) == 4) {
        mealCount++;
    }
    fclose(file);
    printf("Data loaded successfully.\n");
}

void to_lower_case(char *str) {
    for (int i = 0; str[i]; i++) {
        str[i] = tolower(str[i]);
    }
}

void addMeal() {
	
	//Check if maximum number of meals have been reached
    if (mealCount >= MAX_MEALS) {	
        printf("Error: Database is full. Cannot add more meals.\n");
        return;	//Exit the function if the limit is reached
    }

    Meal newMeal;	//Declare a newMeal object to store details
    clearInputBuffer();

    printf("Enter meal name: ");
    if (fgets(newMeal.name, MAX_NAME_LEN, stdin) == NULL) {
        printf("Error reading meal name.\n");
        return;
    }
    newMeal.name[strcspn(newMeal.name, "\n")] = '\0';

    printf("Enter category (e.g., breakfast, lunch, snack, dinner): ");
    if (fgets(newMeal.category, MAX_NAME_LEN, stdin) == NULL) {
        printf("Error reading category.\n");
        return;
    }
    newMeal.category[strcspn(newMeal.category, "\n")] = '\0';

    printf("Enter calorie count: ");
    if (scanf("%d", &newMeal.calories) != 1) {
        printf("Invalid input for calorie count.\n");
        clearInputBuffer();
        return;
    }
    clearInputBuffer();

     // Loop until a valid dietary type is entered
    const char *validDietaryTypes[] = {"vegetarian", "non-vegetarian", "vegan"};
    int isValid = 0;

    while (!isValid) {
        printf("Enter dietary type (e.g., vegetarian, non-vegetarian, vegan): ");
        if (fgets(newMeal.dietaryType, MAX_NAME_LEN, stdin) == NULL) {
            printf("Error reading dietary type.\n");
            return;
        }
        newMeal.dietaryType[strcspn(newMeal.dietaryType, "\n")] = '\0';
        
        // Convert the input dietary type to lower case
        to_lower_case(newMeal.dietaryType);

        // Check if the entered dietary type is valid
        for (int i = 0; i < sizeof(validDietaryTypes) / sizeof(validDietaryTypes[0]); i++) {
        	 // Create a copy of the valid dietary type and convert it to lower case for comparison
            char validTypeLower[MAX_NAME_LEN];
            strncpy(validTypeLower, validDietaryTypes[i], MAX_NAME_LEN);
            validTypeLower[MAX_NAME_LEN - 1] = '\0';
            to_lower_case(validTypeLower);

            if (strcmp(newMeal.dietaryType, validTypeLower) == 0) {
                isValid = 1; // Valid dietary type found
                break;
            }
        }

        if (!isValid) {
            printf("Invalid dietary type. Please enter one of the following: vegetarian, non-vegetarian, vegan.\n");
        }
    }
    
    printf("Valid dietary type entered: %s\n", newMeal.dietaryType);

    mealDatabase[mealCount++] = newMeal;	//Add the new meal to the array of meals at the current meal position
    saveToFile();
    printf("Meal added and saved successfully.\n");
}

void displayMeals() {
    if (mealCount == 0) {
        printf("No meals found in the database.\n");
        return;
    }

    printf("\nMeal Database:\n");
    for (int i = 0; i < mealCount; i++) {
        printf("%d. Name: %s, Category: %s, Calories: %d, Dietary Type: %s\n",
               i + 1, mealDatabase[i].name, mealDatabase[i].category, mealDatabase[i].calories, mealDatabase[i].dietaryType, mealDatabase[i].dietaryType);
    }
}

void deleteMeal() {
    int index;
    displayMeals();

    printf("Enter the meal number to delete: ");
    if (scanf("%d", &index) != 1 || index < 1 || index > mealCount) {
        printf("Invalid meal number.\n");
        return;
    }

    for (int i = index - 1; i < mealCount - 1; i++) {
        mealDatabase[i] = mealDatabase[i + 1];
    }
    mealCount--;
    saveToFile();
    printf("Meal deleted successfully.\n");
}

void updateMeal() {
	//Search for the meal to update
    int index;
    displayMeals();	//Function call to display meals

	//Prompt user to update new details
    printf("\n\nEnter the meal number to update: ");
    if (scanf("%d", &index) != 1 || index < 1 || index > mealCount) {
        printf("Invalid meal number.\n");
        return;
    }

    Meal *meal = &mealDatabase[index - 1];
    clearInputBuffer();

    printf("Enter new meal name (current: %s): ", meal->name);
    if (fgets(meal->name, MAX_NAME_LEN, stdin) != NULL) {
        meal->name[strcspn(meal->name, "\n")] = '\0';
    }

    printf("Enter new category (current: %s): ", meal->category);
    if (fgets(meal->category, MAX_NAME_LEN, stdin) != NULL) {
        meal->category[strcspn(meal->category, "\n")] = '\0';
    }

    printf("Enter new calorie count (current: %d): ", meal->calories);
    //If the calorie count is invalid, display error message and return
    if (scanf("%d", &meal->calories) != 1) {
        printf("Invalid input for calorie count.\n");
        clearInputBuffer();
        return;
    }
    clearInputBuffer();

    printf("Enter new dietary type (current: %s): ", meal->dietaryType);
    if (fgets(meal->dietaryType, MAX_NAME_LEN, stdin) != NULL) {
        meal->dietaryType[strcspn(meal->dietaryType, "\n")] = '\0';
    }

    saveToFile();
    printf("Meal updated successfully.\n");
}

void calculateCaloricRequirement() {
    int age, weight, height, activityLevel, goal;
    char gender;
    float bmr, dailyCaloricRequirement;

    printf("Enter age: ");
    scanf("%d", &age);

    printf("Enter gender (M/F): ");
    scanf(" %c", &gender);
    
    printf("Enter weight (kg): ");
    scanf("%d", &weight);

    printf("Enter height (cm): ");
    scanf("%d", &height);

    printf("Enter activity level (1: Sedentary, 2: Moderate, 3: Active): ");
    scanf("%d", &activityLevel);

    printf("Enter health goal (1: Weight loss, 2: Weight gain): ");
    scanf("%d", &goal);

    if (gender == 'M' || gender == 'm') {
        bmr = 10 * weight + 6.25 * height - 5 * age + 5;
    } else if (gender == 'F' || gender == 'f') {
        bmr = 10 * weight + 6.25 * height - 5 * age - 161;
    } else {
        printf("Invalid gender input. Please enter 'M' or 'F'.\n");
        return;
    }

//selection for user activeness
    switch (activityLevel) { 
        case 1: //type of selection
            dailyCaloricRequirement = bmr * 1.2;
            break; // it stops from looping
        case 2:
            dailyCaloricRequirement = bmr * 1.55;
            break;
        case 3:
            dailyCaloricRequirement = bmr * 1.9;
            break;
        default:
            printf("Invalid activity level. Please enter 1, 2, or 3.\n");
            return;
    }

    if (goal == 1) {
        dailyCaloricRequirement -= 500;
    } else if (goal == 2) {
        dailyCaloricRequirement += 500;
    } else {
        printf("Invalid health goal. Please enter 1 or 2.\n");
        return;
    }
	calculatedCalorieTarget = (int)dailyCaloricRequirement; // Store the calorie target
    printf("Your daily caloric requirement is: %.2f calories\n", dailyCaloricRequirement);
}

void saveMealHistory(const char *filename, const char *mealPlan) { 
    FILE *file = fopen(filename, "a"); 
    if (file) { 
        fprintf(file, "%s\n", mealPlan); 
        fclose(file);
    }
}


void generateMealPlan(int day) {
    int calorieTarget = calculatedCalorieTarget; // Use the stored calorie target as default
    printf("Enter your desired calorie amount (current target is %d): ", calorieTarget);
    scanf("%d", &calorieTarget);

    printf("Generated Meal Plan for Day %d:\n", day + 1);
    char mealPlan[1000] = "";
    int totalCalories = 0;
    int categoryCount[4] = {0}; // [0]: breakfast, [1]: lunch, [2]: dinner, [3]: snack
    int i, addedMeals = 0;

    // Pass 1: Ensure one meal from each category
    for (i = 0; i < mealCount && totalCalories < calorieTarget; i++) {
    	char category[50];
        strcpy(category, mealDatabase[i].category);
        to_lower_case(category);
        
        if (categoryCount[0] == 0 && strcmp(mealDatabase[i].category, "breakfast") == 0) {
            char mealDetails[256];
            sprintf(mealDetails, "Meal: %s, Category: %s, Calories: %d\n", mealDatabase[i].name, mealDatabase[i].category, mealDatabase[i].calories);
            strcat(mealPlan, mealDetails);
            totalCalories += mealDatabase[i].calories;
            categoryCount[0]++;
            addedMeals++;
        } else if (categoryCount[1] == 0 && strcmp(mealDatabase[i].category, "lunch") == 0) {
            char mealDetails[256];
            sprintf(mealDetails, "Meal: %s, Category: %s, Calories: %d\n", mealDatabase[i].name, mealDatabase[i].category, mealDatabase[i].calories);
            strcat(mealPlan, mealDetails);
            totalCalories += mealDatabase[i].calories;
            categoryCount[1]++;
            addedMeals++;
        } else if (categoryCount[2] == 0 && strcmp(mealDatabase[i].category, "dinner") == 0) {
            char mealDetails[256];
            sprintf(mealDetails, "Meal: %s, Category: %s, Calories: %d\n", mealDatabase[i].name, mealDatabase[i].category, mealDatabase[i].calories);
            strcat(mealPlan, mealDetails);
            totalCalories += mealDatabase[i].calories;
            categoryCount[2]++;
            addedMeals++;
        } else if (categoryCount[3] == 0 && strcmp(mealDatabase[i].category, "snack") == 0) {
            char mealDetails[256];
            sprintf(mealDetails, "Meal: %s, Category: %s, Calories: %d\n", mealDatabase[i].name, mealDatabase[i].category, mealDatabase[i].calories);
            strcat(mealPlan, mealDetails);
            totalCalories += mealDatabase[i].calories;
            categoryCount[3]++;
            addedMeals++;
        }
    }

    // Pass 2: Add more meals to meet the desired calorie amount
    while (totalCalories < calorieTarget) {
        for (i = 0; i < mealCount && totalCalories < calorieTarget; i++) {
        	char category[50];
            strcpy(category, mealDatabase[i].category);
            to_lower_case(category);
            
            char mealDetails[256];
            sprintf(mealDetails, "Meal: %s, Category: %s, Calories: %d\n", mealDatabase[i].name, mealDatabase[i].category, mealDatabase[i].calories);
            strcat(mealPlan, mealDetails);
            totalCalories += mealDatabase[i].calories;
            if (strcmp(mealDatabase[i].category, "breakfast") == 0) categoryCount[0]++;
            else if (strcmp(mealDatabase[i].category, "lunch") == 0) categoryCount[1]++;
            else if (strcmp(mealDatabase[i].category, "dinner") == 0) categoryCount[2]++;
            else if (strcmp(mealDatabase[i].category, "snack") == 0) categoryCount[3]++;
            addedMeals++;
        }
    }

    printf("%s", mealPlan);
    printf("Total Calories in Meal Plan: %d\n", totalCalories);
    dailyCalories[day] = totalCalories;
    saveMealHistory("meal_history.txt", mealPlan);

    // Update weekly category count
    for (i = 0; i < 4; i++) {
        weeklyCategoryCount[i] += categoryCount[i];
    }

    // Display total meals per category
    printf("\nTotal Meals Planned per Category for Day %d:\n", day + 1);
    printf("Breakfast: %d\n", categoryCount[0]);
    printf("Lunch: %d\n", categoryCount[1]);
    printf("Dinner: %d\n", categoryCount[2]);
    printf("Snack: %d\n", categoryCount[3]);
}

void viewMealHistory() {
    FILE *file = fopen("meal_history.txt", "r"); 
    if (file) { 
        char line[256]; 
        printf("Meal History:\n"); 
        while (fgets(line, sizeof(line), file)) { 
            printf("%s", line); 
        } 
        fclose(file);
    } else { 
        printf("No meal history found.\n");
    }
}

void displayMenu() {
	//Display main menu to user
    printf("\n--- Smart Meal Planner System ---\n");
    printf("1. Meal Database Management\n");
    printf("2. Calculate Caloric Requirement\n");
    printf("3. Generate Meal Plan\n");
    printf("4. View Meal History\n");
    printf("5. Display Weekly Summary\n");
    printf("6. Exit\n");
}

void displayWeeklySummary() {
    int totalWeeklyCalories = 0;
    printf("\n--- Weekly Caloric Summary ---\n");
	int day;
    for (day = 0; day < DAYS; day++) {
        printf("Day %d: %d calories\n", day + 1, dailyCalories[day]);
        totalWeeklyCalories += dailyCalories[day];

        // Check if calorie target was exceeded
        if (dailyCalories[day] > calculatedCalorieTarget) {
            printf("**Alert**: Calorie target exceeded on Day %d.\n", day + 1);
        }
    }
    
    float weeklyAverageCalories = totalWeeklyCalories / (float)DAYS;
    printf("Total Weekly Calories: %d calories\n", totalWeeklyCalories);
    printf("Weekly Average Calories: %.2f calories\n", weeklyAverageCalories);

    // Display total meals per category
    printf("\nTotal Meals Planned per Category for the Week:\n");
    printf("Breakfast: %d\n", weeklyCategoryCount[0]);
    printf("Lunch: %d\n", weeklyCategoryCount[1]);
    printf("Dinner: %d\n", weeklyCategoryCount[2]);
    printf("Snack: %d\n", weeklyCategoryCount[3]);
}

//Main function begins execution
int main() {
    loadFromFile();

    int choice;
    do {
    	//Display menu for user
        displayMenu();
        printf("Enter your choice: ");
        if (scanf("%d", &choice) != 1) {	//Check if the input is a digit and if the integer is between 1 and 7
            printf("Invalid input. Please enter a number.\n");	// If the input is invalid, prompt the user to enter a valid number 
            clearInputBuffer();
            continue;
        }

        switch (choice) {
            case 1:
                printf("\n--- Manage Meal Database ---\n");
                displayMeals();

                int subChoice;
                //Display sub-menu for user
                while (1) {
                    printf("\n1. Add Meal\n");
                    printf("2. Update Meal\n");
                    printf("3. Delete Meal\n");
                    printf("4. Return to Main Menu\n");
                    printf("Enter your choice: ");
                    if (scanf("%d", &subChoice) != 1) {	//Check if the input is a digit and if the integer is between 1 and 5
                        printf("Invalid input. Please enter a number.\n");
                        while (getchar() != '\n');  // Clear the input buffer
                        continue;
                    }

                    switch (subChoice) {
                        case 1:
                            addMeal();
                            displayMeals();
                            break;
                        case 2:
                            updateMeal();
                            displayMeals();
                            break;
                        case 3:
                            deleteMeal();
                            displayMeals();
                            break;
                        case 4:
                        	printf("Returning to main menu.\n");
                       		goto displayMenu;
                        default:
                            printf("Invalid choice. Please try again.\n");
                    }
                }
            displayMenu:
                break;
            case 2:
                calculateCaloricRequirement();
                break;
            case 3: { 
				int day; 
				printf("Enter day of the week (1-7): "); 
				scanf("%d", &day); 
				generateMealPlan(day - 1); 
				break; 
			}
            case 4:
                viewMealHistory();
                break;
            case 5:
                displayWeeklySummary();
                break;
            case 6:
				printf("Exiting the system. Goodbye!\n");
                break;
            default:
                printf("Invalid choice. Please try again.\n");
        }
    } while (choice != 6);	//End the looping when user selects option 7
    return 0;	//Indicates successful program termination
}	//End main
