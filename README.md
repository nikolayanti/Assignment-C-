#include <iostream>
#include <fstream>
#include <iomanip>
#include <string>
#include <cctype>
using namespace std;

const int MaxProducts = 100;

class InventorySystem {
private:
    string ID[MaxProducts];
    string Product_Name[MaxProducts];
    string Category[MaxProducts];
    double Price[MaxProducts];
    int Quantity[MaxProducts];
    int Reorder_level[MaxProducts];
    int Product_count;

  /* Load data from text file
       File format: ID Product_Name(no space) Category(no space) Price Quantity Reorder-level
       Example:  P001 Mister_Potato Snacks 4.5 49 15
       Note: Product name and Category cannot include space (use underscore to replace space)
       Data validation:
       1. Price and Quantity should be positive number
       2. File must contain at least 30 different products */
    void load_from_file() {
        const int MinProducts = 30; // Minimum products require
        ifstream inputFile;
        inputFile.open("Inventory.txt");

        // a. Check if the file can be opened
        if(!inputFile) {
            cout << "Error opening the file.\n";
            exit(1); // Abnormal program termination
        }

        // b. Initialize the number of products
        Product_count = 0;

        // c. Read  data line by line
        while (Product_count < MaxProducts
            && inputFile >> ID[Product_count] >> ws // Read the product ID (skip the white space)
            && getline(inputFile, Product_Name[Product_count], ' ') // Read the product name (end by space)
            && getline(inputFile, Category[Product_count], ' ') // Read the product's category (end by space)
            && inputFile >> Price[Product_count] >> Quantity[Product_count] >> Reorder_level[Product_count]) {
            // Check for non-negative prices and quantity
            if (Price[Product_count] < 0 || Quantity[Product_count] < 0) {
                cout << "Error: invalid data at line " << Product_count+1 << "(Price/Quantity should be positive)." << endl;
                exit(1);
            }
            Product_count++; // Count only when the data is legal
        }

        // d. Verify minimum product requirement
        if (Product_count < MinProducts) {
            cout << "Error: The file must include " << MinProducts << " different products, but only " << Product_count << " were found!" << endl;
            inputFile.close();
            exit(1);
        }

        inputFile.close();
    }

public:
    /* InventorySystem class constructor
       Features:
        1. Iniatializes Product_count = 0
        2. Automatically calls load_from_file() to load the data
       Note: If fail to load the file, Product_count remains 0 */
    InventorySystem() : Product_count(0) {
        load_from_file();
    }

    /* User Authentication
       Username = "Grocery Store" and Password = "gro12345"
       Safety mechanism: After 3 failures, forced exit the program*/
    void login() {
        string username, password;
        int attempts = 0;
        const int Max_attempts = 3;
        while (attempts < Max_attempts) {
            cout << "===== LOGIN SYSTEM =====" << endl;
            cout << "Username: ";
            getline(cin, username); // Use getline to prevent space truncation
            cout << "Password: ";
            getline(cin, password);
            if (username == "Grocery Store" && password == "gro12345") {
                cout << "\nLogin successful!" << endl;
                break;
            }
            else {
                attempts++; // Track failed attempts
                if (attempts < Max_attempts) { // Maximum 3 attempts
                    cout << "Username or Password invalid! Remaining attempts: " << 3 - attempts << endl << endl;
                }
                else {
                    cout << "Login failed! System exiting..." << endl;
                    exit(0); // Immediate termination of the program
                }
            }
        }
    }

    /* Main menu control panel interface
       Features:
        a. Cycle display the 7 options menu
        b. Processes only integer inputs from 1 to 7
        c. Option 7 triggers save and exit the program */
    void display_menu() {
        int choice;
        do {
            cout << "\n===== STOCK INVENTORY MANAGEMENT SYSTEM =====" << endl;
            cout << "1. List all products" << endl;
            cout << "2. View a particular product" << endl;
            cout << "3. Add a product" << endl;
            cout << "4. Delete product(s)" << endl;
            cout << "5. Search product(s)" << endl;
            cout << "6. Update a product" << endl;
            cout << "7. Save and Exit" << endl;
            cout << "Enter your choice: ";
            cin >> choice;
            // Switch to the corresponding operations
            switch(choice) {
            case 1:
                list_Product(); // Calls the function to view all the products' details
                break;
            case 2:
                view_Product(); // Calls the function to view product details
                break;
            case 3:
                add_Product(); // Calls the function to add product to the list of existing products
                break;
            case 4:
                delete_Product(); // Calls the function to delete product(s)
                break;
            case 5:
                search_Product(); // Calls the function to search data
                break;
            case 6:
                update_Product(); // Calls the function to update a product
                break;
            case 7:
                cout << "\nSaving data and exiting..." << endl;
                save_exit(); // Calls function to save data and exit program
                return;
            default:
                cout << "Invalid choice! Try again." << endl; // Handles invalid integer inputs
                cin.clear(); // Clears the error state of cin
                cin.ignore(100, '\n'); // Skip the invalid input
                break;
            }
        } while(choice != 7); // Loops until user selects exit (Option 7)
    }

    // 1. List all products
    void list_Product() {
        //checking empty inventory
        if (Product_count == 0) {
            cout << "\nNo products in inventory!" << endl;
            return;
        }

        //display table for inventory tracking
        cout << "\n============================================= INVENTORY TRACKING ================================================" << endl;
        cout << setw(8) << "ID" << setw(20) << "Name" << setw(20) << "Category" << setw(15) << "Price(RM)" << setw(15) << "Quantity" << setw(20) << "Re-order Level" << setw(15) << "Status" << endl;
        cout << "-----------------------------------------------------------------------------------------------------------------" << endl;


        /*display each product's data */
        for (int i = 0; i < Product_count; i++) {
            cout << setw(8) << ID[i]
            << setw(20) << Product_Name[i]
            << setw(20) << Category[i]
            << setw(15) << fixed << setprecision(2) << Price[i]
            << setw(15) << Quantity[i]
            << setw(20) << Reorder_level[i];

        //check low stock alert
        if (Quantity[i] < Reorder_level[i]) {
            cout << setw(15) << "LOW STOCK"; }
        cout << endl;
        }
    }

    /* 2. View a particular product
        a. View product details for the specified ID
        b. Low inventory warning (triggers when quantity lower than re-order level) */
    void view_Product() {

        // Check for products in current inventory
        if(Product_count == 0) {
            cout << "\nThere are currently no products in stock!" << endl;
            return;
        }

        string viewID;
        cout << "\n========== VIEW A PARTICULAR PRODUCT ==========" << endl;
        cout << "Enter the product ID you want to view: ";
        cin.ignore();
        getline(cin, viewID);

        // Check if the product ID is in the list
        bool found = false;
        for(int i = 0; i < Product_count; i++) {
            if (ID[i] == viewID) { // If the input ID matches with the specified ID, display the product details
                cout << "\n===== PRODUCT DETAIL =====" << endl;
                cout << "ID: " << ID[i] << endl;
                cout << "Name: " << Product_Name[i] << endl;
                cout << "Category: " << Category[i] << endl;
                cout << "Price: RM" << fixed << setprecision(2) << Price[i] << endl; // Fixed the price output to two decimal places
                cout << "Quantity: " << Quantity[i] << endl;
                cout << "Re-order Level: " << Reorder_level[i] << endl;
                if (Quantity[i] < Reorder_level[i]) {
                    cout << "\nWarning: Stock is below restock level!"; // Output alert message when stock falls below re-order level
                }
                found = true;
                break;
            }
        }

        if(!found) {
            cout << "Product with ID \'" << viewID << "\' not found!" << endl; // Output error message when user inputs invalid ID
        }
    }

    /* 3. Add a new product data to the list of existing product data extracted from read text file :
      a. Firstly, request for new ID of accepted format ("P034") and store it in the array
      b. Secondly, input the information required about the new product, and store it in the consecutive array spaces
      c. Input validation for new data entered, except product name
      d. Increment the total array spaces number and store the new data into the new space */
    void add_Product() {

        // Check if current product amount does not exceed the maximum amount (100)
        if (Product_count >= MaxProducts) {
            cout << "\nInventory is full, you can't add more than " << MaxProducts << " products!" << endl;
            return;
        }

        // The "add_" label for naming convention
        string add_id, add_name, add_category;
        double add_price;
        int add_quantity, add_reorder;

        cin.ignore();  // Flush input buffer
        cout << "\n========== ADD A PRODUCT ==========" << endl;

        // Set the ID to required format as in the source text file for safe sorting
        while (true) {
            cout << "Enter Product ID, format should be (P###): ";
            getline(cin, add_id);

            if (add_id.length() == 4 && add_id[0] == 'P' && isdigit(add_id[1]) && isdigit(add_id[2]) && isdigit(add_id[3])) { // Checks if ID ifs of format "P033"
                int idNumber = stoi(add_id.substr(1));
                if (idNumber < MaxProducts) break;
            else
                cout << "Error: Product ID number must be less than " << MaxProducts << " (e.g., P099).\n"; // Checking if entered ID is not higher than 100, for sorting convention
        }
        else
            cout << "Invalid ID format. Please enter ID in format P followed by 3 digits (e.g., P030).\n";
    }

        // Checking if ID is already exists
        for (int i = 0; i < Product_count; i++) {
            if (ID[i] == add_id) {
                cout << "Error: Product ID already exists!" << endl;
                return;
            }
        }

        // No validation for the product name, as name "iPhone_15" is accepted
        cout << "Enter Product Name: ";
        getline(cin, add_name);

        // Allowed product categories, that user can add to in the supermarket system
        const string allowedCategories[] =
        { "Snacks", "Beverages", "Grocery", "Meat", "Personal_Care","Electronics", "Sports", "Clothing", "Stationery", "Dairy", "FrozenFood", "Seafood", "Appliances", "Household", "Toys","Furniture", "Books", "Garden", "Tools", "Baby", "Health","Pet_Supplies", "Automotive", "Beauty", "Office_Supplies"};

        /* Listing out all allowed categories for the product and add according to them, if input is incorrect input again until correct
        */
        while (true) {
            cout << "\nAllowed Categories:\n";
            for (const string& cat : allowedCategories) // List out all the categories for user
                cout << "- " << cat << endl;

            cout << "\nEnter Product Category (enter it exactly as shown above): ";
            getline(cin, add_category);

            bool valid = false;
            for (const string& category : allowedCategories) {
                if (add_category == category) {
                    valid = true;
                    break;
                }
            }

            if (!valid) {
                cout << "Invalid category. Input again.\n";
            } else break;
        }

        // Product price validation, while loop runs to get the non-character, non-negative number
        while (true) {
            cout << "Enter Product Price: RM";
            cin >> add_price;

            if (cin.fail() || add_price < 0) { // Check price contain characters or negative number
                cin.clear();
                cin.ignore(100, '\n');
                cout << "Invalid input. Price must be a non-negative number.\n"; // loop will run infinitely until required data received
            } else break;
        }

        // Product quantity validation, same condition as set for price
        while (true) {
            cout << "Enter Quantity on hand: ";
            cin >> add_quantity;

            if (cin.fail() || add_quantity < 0) {
                cin.clear();
                cin.ignore(100, '\n');
                cout << "Invalid input. Quantity must be a non-negative number.\n";
            } else break;
        }

        // Reorder level validation, same as other numeric data type validations in this function
        while (true) {
            cout << "Enter Re-order level: ";
            cin >> add_reorder;

            if (cin.fail() || add_reorder < 0) {
                cin.clear();
                cin.ignore(100, '\n');
                cout << "Invalid input. Reorder level must be a non-negative integer.\n";
            } else break;
        }

        // Store the values in the consecutive array spaces
        ID[Product_count] = add_id;
        Product_Name[Product_count] = add_name;
        Category[Product_count] = add_category;
        Price[Product_count] = add_price;
        Quantity[Product_count] = add_quantity;
        Reorder_level[Product_count] = add_reorder;
        Product_count++; // Increment the count of the products, if new one is successfully added

        cout << "Product added successfully!" << endl;
    }

    // 4. Delete product(s)
    void delete_Product() {
        if (Product_count == 0){
            cout << "\nInventory is empty, no products to delete!" << endl;
            return;
        }

        cin.ignore();
        string deleteID;
        cout << "\n========== DELETE PRODUCT ==========" << endl;
        cout << "Enter the Product ID to delete: ";
        getline(cin, deleteID);

        bool found = false;
        for (int i = 0; i < Product_count; i++) {
            if (ID[i] == deleteID) {
            // Shift products to fill the gap
                for (int j = i; j < Product_count - 1; j++) {
                    ID[j] = ID[j + 1];
                    Product_Name[j] = Product_Name[j + 1];
                    Category[j] = Category[j + 1];
                    Price[j] = Price[j + 1];
                    Quantity[j] = Quantity[j + 1];
                    Reorder_level[j] = Reorder_level[j + 1];
                }
            Product_count--;
            found = true;
            cout << "Product deleted successfully!" << endl;
            break;
            }
        }

        if (!found)  {
            cout << "Product with ID '" << deleteID << "' not found!" << endl;
        }
    }

   // 5. Search products(s)
    void search_Product() {

        // Check for products in current inventory
        if(Product_count == 0) {
            cout << "\nThere are currently no products in stock!" << endl;
            return;
        }

        // Search for products by name,category or unique ID & filter products based on stock levels or price range
        int searchpro;
        string searchname, searchcategory, searchID;
        do {
            cout << "\n========== SEARCH PRODUCT(S) ==========";
            cout << "\n1. Name" << endl;
            cout << "2. Category" << endl;
            cout << "3. ID" << endl;
            cout << "4. Stock levels" << endl;
            cout << "5. Price range" << endl;
            cout << "6. Back to Main Menu" << endl;
            cout << "Please select an action: ";
            cin >> searchpro;

            // Switch to the corresponding action
            switch(searchpro) {
            case 1: {
                // Search by Name
                cout << "Enter product name: ";
                cin.ignore();
                getline(cin, searchname);
                bool found = false;
                // // Loops through the data to display the product with the product name
                for (int i = 0; i < Product_count; i++) {
                    if (Product_Name[i] == searchname) {
                        cout << "\n================================================= SEARCH BY NAME =================================================" << endl;
                        cout << setw(8) << "ID" << setw(20) << "Name" << setw(20) << "Category" << setw(15) << "Price (RM)" << setw(15) << "Quantity" << setw(20) << "Re-order Level" << setw(15) << "Status" << endl;
                        cout << "-----------------------------------------------------------------------------------------------------------------" << endl;
                        cout << setw(8) << ID[i] << setw(20) << Product_Name[i] << setw(20) << Category[i] << setw(15) << fixed << setprecision(2) << Price[i] << setw(15) << Quantity[i] << setw(20) << Reorder_level[i];
                        // Low stock warning
                        if (Quantity[i] < Reorder_level[i]) {
                            cout << setw(15) << "LOW STOCK";
                        }
                        found = true;
                    }
                }
                cout << endl;

                if(!found) {
                    cout << "Product with the name \'" << searchname << "\' not found!" << endl;
                }
                break;
            }
            case 2: {
                // Search by Category
                cout << "Enter product Category: ";
                cin.ignore();
                getline(cin, searchcategory);
                bool found = false;
                // Loops through the data to display products with user-entered categories
                for (int i = 0; i < Product_count; i++) {
                    if (Category[i] == searchcategory) {
                        cout << "\n=============================================== SEARCH BY CATEGORY ===============================================" << endl;
                        cout << setw(8) << "ID" << setw(20) << "Name" << setw(20) << "Category" << setw(15) << "Price (RM)" << setw(15) << "Quantity" << setw(20) << "Re-order Level" << setw(15) << "Status" << endl;
                        cout << "-----------------------------------------------------------------------------------------------------------------" << endl;
                        cout << setw(8) << ID[i] << setw(20) << Product_Name[i] << setw(20) << Category[i] << setw(15) << fixed << setprecision(2) << Price[i] << setw(15) << Quantity[i] << setw(20) << Reorder_level[i];
                        // Low Stock warning
                        if (Quantity[i] < Reorder_level[i]) {
                            cout << setw(15) << "LOW STOCK";
                        }
                        found = true;
                    }
                }
                cout << endl;

                if(!found) {
                    cout << "Product with the category \'" << searchcategory << "\' not found!" << endl;
                }
                break;
            }
            case 3: {
                // Search by ID
                cout << "Enter product ID: ";
                cin.ignore();
                getline(cin, searchID);
                bool found = false;
                // Loops through the data to display the product with the ID
                for (int i = 0; i < Product_count; i++) {
                    if (ID[i] == searchID) {
                        cout << "\n================================================== SEARCH BY ID ==================================================" << endl;
                        cout << setw(8) << "ID" << setw(20) << "Name" << setw(20) << "Category" << setw(15) << "Price (RM)" << setw(15) << "Quantity" << setw(20) << "Re-order Level" << setw(15) << "Status" << endl;
                        cout << "-------------------------------------------------------------------------------------------------------------------" << endl;
                        cout << setw(8) << ID[i] << setw(20) << Product_Name[i] << setw(20) << Category[i] << setw(15) << fixed << setprecision(2) << Price[i] << setw(15) << Quantity[i] << setw(20) << Reorder_level[i];
                        // Low stock warning
                        if (Quantity[i] < Reorder_level[i]) {
                            cout << setw(15) << "LOW STOCK";
                        }
                        found = true;
                    }
                }
                cout << endl;

                if(!found) {
                    cout << "Product with the ID \'" << searchID << "\' not found!" << endl;
                }
                break;
            }
            case 4: {
                // Filter products based on stock levels
                int threshold;

                // Input validation
                while(true) {
                    cout << "Enter maximum stock level to filter: ";
                    cin >> threshold;
                    if(cin.fail() || threshold<0) {
                        cin.clear();
                        while(cin.get() != '\n');
                        cout << "Invalid input. PLease enter a positive integer." << endl;
                    }
                    else {
                        break; }
                    }

                // Display Header
                cout << "\n========================================= ITEMS WITH STOCK LESS THAN " << threshold << " =========================================" << endl;
                cout << setw(8) << "ID" << setw(20) << "Name" << setw(20) << "Category" << setw(15) << "Price (RM)" << setw(15) << "Quantity" << setw(20) << "Re-order Level" << setw(15) << "Status" << endl;
                cout << "-----------------------------------------------------------------------------------------------------------------" << endl;

                bool found=false;

                // Loop through all the products
                for(int i=0; i < Product_count; i++) {
                    if(Quantity[i] <= threshold) {
                        cout << setw(8) << ID[i] << setw(20) << Product_Name[i] << setw(20) << Category[i] << setw(15) << fixed << setprecision(2) << Price[i] << setw(15) << Quantity[i] << setw(20) << Reorder_level[i];

                    // Low stock warning
                    if(Quantity[i] < Reorder_level[i]) {
                        cout << setw(15) << "LOW STOCK"; }
                    cout << endl;
                    found = true; }
                }

                // If no products matches the filter
                if(!found) {
                    cout << "No items found." << endl; }
                break;
            }
            case 5: {
                // Filter products based on price range
                double minPrice, maxPrice;

                // Min price
                while(true) {
                    cout << "Enter minimum price: RM";
                    cin >> minPrice;
                    if(cin.fail() || minPrice < 0) {
                        cin.clear();
                        while(cin.get() != '\n');
                        cout << "Invalid input. Please enter a positive number." << endl;
                    }
                    else {
                        break; }
                }

                // Max price
                while(true) {
                    cout << "Enter maximum price: RM";
                    cin >> maxPrice;
                    if(cin.fail() || maxPrice < minPrice) {
                        cin.clear();
                        while(cin.get() != '\n');
                        cout << "Invalid input. Please enter price more than RM" << minPrice << "." << endl;
                    }
                    else {
                        break; }
                }

                // Display Header
                cout << "\n============================================= PRODUCTS (RM" << minPrice << " - RM" << maxPrice << ") =============================================" << endl;
                cout << setw(8) << "ID" << setw(20) << "Name" << setw(20) << "Category" << setw(15) << "Price (RM)" << setw(15) << "Quantity" << setw(20) << "Re-order Level" << setw(15) << "Status" << endl;
                cout << "-----------------------------------------------------------------------------------------------------------------" << endl;

                bool found = false;
                for(int i = 0; i < Product_count; i++) {
                    if(Price[i] >= minPrice && Price[i] <= maxPrice) {
                        cout << setw(8) << ID[i] << setw(20) << Product_Name[i] << setw(20) << Category[i] << setw(15) << fixed << setprecision(2) << Price[i] << setw(15) << Quantity[i] << setw(20) << Reorder_level[i];

                    // Low stock warning
                    if(Quantity[i] < Reorder_level[i]) {
                        cout << setw(15) << "LOW STOCK"; }
                    cout << endl;
                    found = true; }
                }

                if(!found) {
                    cout << "No products found in this price range." << endl; }
                break;
            }
             case 6:
                return; // Exit to the main menu
            default:
                cout << "Invalid choice!" << endl; // Handles invalid integer inputs
                cin.clear(); // Clears the error state of cin
                cin.ignore(1000, '\n'); //Skips the invalid input
                break;
            }
        } while (searchpro != 6); // Loops until user selects back to main menu (Option 6)
    }

    // 6. Updating product information
    void update_Product() {
        if (Product_count == 0) {
            cout << "\nNo products to update!" << endl;
            return;
        }

        cin.ignore();
        string updateID;
        cout << "\n========== UPDATE PRODUCT ==========" << endl;
        cout << "Enter the Product ID to update: ";
        getline(cin, updateID);

        bool found = false;
        for (int i = 0; i < Product_count; i++) {
            if (ID[i] == updateID) {
                found = true;

                int choice;
                do {
                    cout << "\nWhich field do you want to update?" << endl;
                    cout << "1. Product ID" << endl;
                    cout << "2. Name" << endl;
                    cout << "3. Category" << endl;
                    cout << "4. Price" << endl;
                    cout << "5. Quantity" << endl;
                    cout << "6. Reorder Level" << endl;
                    cout << "7. Exit Update Menu" << endl;
                    cout << "Choice: ";
                    cin >> choice;

                    cin.ignore();
                    switch (choice) {
                        case 1: {
                            string newID;
                            cout << "Enter new Product ID: ";
                            getline(cin, newID);
                            ID[i] = newID;
                            break;
                        }
                        case 2: {
                            string newName;
                            cout << "Enter new Product Name: ";
                            getline(cin, newName);
                            Product_Name[i] = newName;
                            break;
                        }
                        case 3: {
                            string newCategory;
                            cout << "Enter new Category: ";
                            getline(cin, newCategory);
                            Category[i] = newCategory;
                            break;
                        }
                        case 4: {
                            double newPrice;
                            while (true)
                            {
                                cout << "Enter new Price: RM";
                                cin >> newPrice;
                                if (cin.fail() || newPrice < 0)
                                {
                                    cin.clear();
                                    cin.ignore(100, '\n');
                                    cout << "Invalid input. Price must be non-negative.\n";
                                }
                                else break;
                            }
                            Price[i] = newPrice;
                            break;
                        }
                        case 5: {
                            int newQty;
                            while (true)
                            {
                                cout << "Enter new Quantity: ";
                                cin >> newQty;
                                if (cin.fail() || newQty < 0)
                                {
                                    cin.clear();
                                    cin.ignore(100, '\n');
                                    cout << "Invalid input. Quantity must be non-negative.\n";
                                }
                                else break;
                            }
                            Quantity[i] = newQty;
                            break;
                        }
                        case 6: {
                            int newReorder;
                            while (true)
                            {
                                cout << "Enter new Reorder Level: ";
                                cin >> newReorder;
                                if (cin.fail() || newReorder < 0)
                                {
                                    cin.clear();
                                    cin.ignore(100, '\n');
                                    cout << "Invalid input. Reorder level must be non-negative.\n";
                                }
                                else break;
                            }
                            Reorder_level[i] = newReorder;
                            break;
                        }
                        case 7: {
                            cout << "Returning to main menu...\n";
                            return;
                        }
                        default: {
                            cout << "Invalid choice!\n";
                            cin.clear();
                            cin.ignore(1000, '\n');
                            break;
                        }
                    }
                } while (choice != 7);
                break;
            }
        }

        if (!found) {
            cout << "Product with ID '" << updateID << "' not found!" << endl;
        }
    }

   /* 7. Save and exit
        a. Save the current inventory data into file
        b. File format: ID Name Category Price(remains 2 decimal places) Quantity Re-order level */
    void save_exit() {
        ofstream outputFile;
        outputFile.open("Inventory.txt");
        if(outputFile) {
            for (int i = 0; i < Product_count; i++) { // Space-delimited format
                outputFile << ID[i] << " " << Product_Name[i] << " " << Category[i]<< " " << fixed << setprecision(2) << Price[i] << " " << Quantity[i] << " " << Reorder_level[i] << endl;
            }
            outputFile.close();
        }
        else {
            cout << "Error opening the file.\n"; // Output error message when fail to open the file
        }
    }
};

/* Main entrance of the program
   Execution process: Construct system object -> login -> enter main menu
   Note: login() fails will directly exit(0), skipping all subsequent processes */
int main() {
    InventorySystem system; // Triggers constructor
    system.login(); // Authentication
    system.display_menu(); // Main loop
    cout << "Program truly ended!" << endl;
    return 0;
}
