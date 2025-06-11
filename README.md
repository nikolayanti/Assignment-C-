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
