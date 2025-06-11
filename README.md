#include <iostream>
#include <fstream>
#include <iomanip>
#include <string>
using namespace std;

// Main menu
void display_menu() {
  cout << "\n===== STOCK INVENTORY MANAGEMENT SYSTEM =====" << endl;
  cout << "1. List all products" << endl;
  cout << "2. View a particular product" << endl;
  cout << "3. Add a product" << endl;    
  cout << "4. Delete product(s)" << endl;
  cout << "5. Search product(s)" << endl;
  cout << "6. Update a product" << endl;
  cout << "7. Save and Exit" << endl;
  cout << "Enter your choice: ";
}

// 7. Save and Exit
void save_exit() {
  ofstream outputFile;
  outputFile.open("Grocery Store Inventory.txt");
  
  if(outputFile) {
    for (int i = 0; i < productCount; i++) {
      outputFile << product[i].getID() << " " << product[i].getProduct_name() << " " << product[i].getPrice() << " " << product[i].getQuantity_on_hand() << " " << product[i].getReorder_level() << endl;
    }
    outputFile.close();
    cout << "Data saved!" << endl;
  }
  else {
    cout << "Error opening the file.\n";
  }
}

// Load from File
void load_from_file() {
  ifstream inputFile;
  inputFile.open("Grocery Store Inventory.txt");

  int ID, Quantity_on_hand, Reorder_level;
  string Product_name;
  double Price;

  if(inputFile) {
    while (inputFile >> ID >> Product_name >> Price >> Quantity_on_hand >> Reorder_level) {
      if (productCount < Max_products) {
        product[productCount++] = Products(ID, Product_name, Price, Quantity_on_hand, Reorder_level);
      }
    }
    inputFile.close();
    cout << "Data loaded!" << endl;
  }
  else {
    cout << "Error opening the file.\n";
  }
}
