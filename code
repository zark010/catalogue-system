#include <stdio.h> 
#include <stdlib.h>
#include <string.h> 
#include <stdbool.h>
#include <stdarg.h>
#include <ctype.h>

#include <openssl/evp.h>
#include <openssl/sha.h>

#define TAX_RATE 0.2 // 20% tax on all items
#define DISCOUNT_THRESHOLD 5 // If you purchase 5 items you get a discount
#define DISCOUNT_ITEMS 0.05 // 5% discount if you purhcase 5 or more items

/* Maximum capacity for arrays */
#define NAME_MAX 70
#define ADDR_MAX 100
#define EMAIL_MAX 60

#define MAX_LINE 256

typedef struct Product Product;
struct Product{
	char *name;
	float price;
	Product *prodNext;
};

typedef struct Customer Customer;
struct Customer{
	char name[NAME_MAX+1];
	char address[ADDR_MAX+1];
	char email[EMAIL_MAX+1];
	Product *cart;
	Customer *cusNext;
};

int generateFileHash(const char *path, unsigned char *hash)
{
	FILE *file = fopen(path, "rb");
	if(!file)
	{
		fprintf(stderr, "Error opening file %s\n", path);
		return -1;
	}

	EVP_MD_CTX *mdctx = EVP_MD_CTX_new();
	if(mdctx == NULL)
	{
		fprintf(stderr, "Error creating MD context\n");
		fclose(file);
		return -1;
	}

	if (EVP_DigestInit_ex(mdctx, EVP_sha256(), NULL) != 1)
    {
        fprintf(stderr, "Error initializing digest.\n");
        EVP_MD_CTX_free(mdctx);
        fclose(file);
        return -1;
    }

    unsigned char buffer[1024];
    int bytesRead;
    while ((bytesRead = fread(buffer, 1, sizeof(buffer), file)) > 0)
    {
        if (EVP_DigestUpdate(mdctx, buffer, bytesRead) != 1)
        {
            fprintf(stderr, "Error updating digest.\n");
            EVP_MD_CTX_free(mdctx);
            fclose(file);
            return -1;
        }
    }

	if (ferror(file))
    {
        fprintf(stderr, "Error reading file: %s\n", path);
        EVP_MD_CTX_free(mdctx);
        fclose(file);
        return -1;
    }

    unsigned int length;
    if (EVP_DigestFinal_ex(mdctx, hash, &length) != 1)
    {
        fprintf(stderr, "Error finalizing digest.\n");
        EVP_MD_CTX_free(mdctx);
        fclose(file);
        return -1;
    }

    EVP_MD_CTX_free(mdctx);
    fclose(file);
    return 0;
}

int compareHashes(const unsigned char *hash1, const unsigned char *hash2, unsigned int length)
{
    return memcmp(hash1, hash2, length) == 0;
}

int writeHashToFile(const char *path, const unsigned char *hash, unsigned int length)
{
    FILE *file = fopen(path, "w");
    if (!file)
    {
        fprintf(stderr, "Error opening hash file: %s\n", path);
        return -1;
    }

    for (unsigned int i = 0; i < length; i++)
    {
        fprintf(file, "%02x", hash[i]);
    }

    fclose(file);
    return 0;
}

int readHashFromFile(const char *path, unsigned char *hash, unsigned int length)
{
    FILE *file = fopen(path, "r");
    if (!file)
    {
        fprintf(stderr, "Error opening hash file: %s\n", path);
        return -1;
    }

    char hexString[2*length + 1];
    if (fread(hexString, 1, 2*length, file) != 2*length)
    {
        fprintf(stderr, "Error reading hash from file: %s\n", path);
        fclose(file);
        return -1;
    }
    hexString[2*length] = '\0';

    for (unsigned int i = 0; i < length; i++)
    {
        sscanf(&hexString[2*i], "%02x", &hash[i]);
    }

    fclose(file);
    return 0;
}

void clearInputBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

Customer * createCustomer(char *name, char *addr, char *email)
{
	Customer *newCus = (Customer *)malloc(sizeof(Customer));
	if(newCus == NULL)
	{
		fprintf(stderr, "Memory allocation for new customer failed.");
		return NULL;
	}

	strncpy(newCus->name, name, NAME_MAX);
	strncpy(newCus->address, addr, ADDR_MAX);
	strncpy(newCus->email, email, EMAIL_MAX);
	newCus->cart = NULL;
	newCus->cusNext = NULL;

	return newCus;
}

// Add customer information (order not important, add at head)
Customer * addCustomer(Customer **head, Customer *newCus)
{
	if(newCus == NULL)
	{
		return *head;
	}
	newCus->cusNext = *head;
	*head = newCus; 
	return *head;
}

// Search method within LL
Customer * findByName(Customer *head, char *name)
{
	Customer *ret = head;
	if(ret == NULL)
	{
		printf("Customer list is empty.\n");
		return NULL;		
	}

	while(ret != NULL)
	{
		if(strcmp(ret->name, name) == 0) {
			return ret;
		}
		ret = ret->cusNext;	
	}
	//printf("Customer %s not found in system--check spelling (case sensitive).\n", name);
	return NULL;
}

// Edit customer information (one parameter at a time)
Customer * editCustomer(Customer *head, char *name, char *what, char *replace)
{
	char param[20];
    strncpy(param, what, 19);
    param[19] = '\0';

    // Convert to lower case
    for (int i = 0; param[i]; i++)
    {
        param[i] = tolower(param[i]);
    }

	// Find customer
	Customer *curr = findByName(head, name);
	if(curr == NULL)
	{
		printf("Customer %s not found, check spelling (case sensitive).\n", name);
		return NULL;
	}

	// Handle what is being edited
	if(strcmp(param, "name") == 0)
	{
		if(strlen(replace) <= NAME_MAX) 
		{
			strncpy(curr->name, replace, NAME_MAX);
			curr->name[NAME_MAX] = '\0';
			printf("Edit successful, customer's new name is: %s.\n", curr->name);
		}
		else {
			printf("Error: New customer name is too long.\n");
			return NULL;
		}
		return head;
	}
	else if(strcmp(param, "address") == 0)
	{
		if(strlen(replace) <= ADDR_MAX) 
		{
			strncpy(curr->address, replace, ADDR_MAX);
			curr->address[ADDR_MAX] = '\0';
			printf("Edit successful, customer's new address is:\n%s\n", curr->address);
		}
		else {
			printf("Error: New address is too long.\n");
			return NULL;
		}
		return head;
	}
	else if(strcmp(param, "email") == 0)
	{
		if(strlen(replace) <= EMAIL_MAX) 
		{
			strncpy(curr->email, replace, EMAIL_MAX);
			curr->email[EMAIL_MAX] = '\0';
			printf("Edit successful, customer's new email is: %s.\n", curr->email);
		}
		else {
			printf("Error: New email is too long.\n");
			return NULL;
		}
		return head;
	}
	else
	{
		printf("Unvalid option to edit. Try inputting one of the following:\n");
		printf("Name\nAddress\nEmail\n");
		return NULL;
	}
}

// delete customer (deleted via name)
Customer * deleteCustomer(Customer *head, char *name)
{
	Customer *prev = NULL;
	Customer *tmp = head;

	// head holds customer to delete
	if(tmp != NULL && strcmp(tmp->name, name) == 0) 
	{
		head = tmp->cusNext;
		free(tmp);
		return head;
	}
	// search for name (can't use prev function since we're handling prev and tmp)
	while(tmp != NULL && (strcmp(tmp->name, name) != 0))
	{
		prev = tmp;
		tmp = tmp->cusNext;
	}
	if(tmp == NULL) {
		printf("Error: Unable to delete customer information, not found.\n");
		return NULL;
	}

	prev->cusNext = tmp->cusNext; // unlink node
	free(tmp);
	//printf("Customer %s deleted successfully.\n", name);
	return head;
}

/* Add products or service: name, price */
Product * createProduct(char *name, float price)
{
	Product *newProd = (Product *)malloc(sizeof(Product));
	if(newProd != NULL)
	{
		newProd->name = (char *)malloc(strlen(name) + 1);
		if(newProd->name != NULL)
		{
			strcpy(newProd->name, name);
			newProd->price = price;
			newProd->prodNext = NULL;
		}
		else
		{
			fprintf(stderr, "Error allocating space for product name.");
			free(newProd);
			newProd = NULL;
		}
	}
	else {
		fprintf(stderr, "Error allocating space for new product.");
	}
	return newProd;
}

// Order not important, insert at head
Product * addProduct(Product **head, Product *newProd)
{
	if(*head == NULL)
	{
		return newProd;
	}
	newProd->prodNext = *head;
	*head = newProd;
	return *head;
}

// Product search
Product * productSearch(Product *head, char *name)
{
	if(head == NULL)
	{
		printf("Catalogue is currently empty, search not executed.\n");
		return head;
	}
	Product *curr = head;
	while(curr != NULL)
	{
		if(strcmp(curr->name, name) == 0) {
			//printf("Product found.\n");
			return curr;
		}
		curr->prodNext;
	}
	printf("Product not found.\n");
	return NULL;
}

Product *productIDSearch(Product *head, int id)
{
	if(head == NULL)
	{
		printf("Catalogue is currently empty, search not executed.\n");
		return head;
	}

	Product *curr = head;
	int i = 1;

	while(curr != NULL)
	{
		if(i == id)
		{
			return curr;
		}
		curr = curr->prodNext;
		i++;
	}
	fprintf(stderr, "Index out of bounds, product not found.\n");
	return NULL;
}

// delete product or service from catalogue
Product * deleteProduct(Product *head, char *name)
{
	Product *prev = NULL;
	Product *tmp = head;

	// head holds customer to delete
	if(tmp != NULL && strcmp(tmp->name, name) == 0) 
	{
		head = tmp->prodNext;
		free(tmp);
		return head;
	}
	// search for name (can't use prev function since we're handling prev and tmp)
	while(tmp != NULL && (strcmp(tmp->name, name) != 0))
	{
		prev = tmp;
		tmp = tmp->prodNext;
	}
	if(tmp == NULL) {
		printf("Error: Unable to delete product, not found.\n");
		return head;
	}

	prev->prodNext = tmp->prodNext; // unlink node
	free(tmp);
	printf("%s successfully deleted from catalogue.\n", name);
	return head;
}

// used for addToCart
Product *createProductCopy(Product *original)
{
    if (original == NULL)
    {
        return NULL;
    }

    Product *copy = (Product *)malloc(sizeof(Product));
    if (copy == NULL)
    {
        fprintf(stderr, "Memory allocation for product copy failed.\n");
        return NULL;
    }

    copy->name = (char *)malloc(strlen(original->name) + 1);
    if (copy->name == NULL)
    {
        fprintf(stderr, "Memory allocation for product name copy failed.\n");
        free(copy);
        return NULL;
    }
    strcpy(copy->name, original->name);

    copy->price = original->price;
    copy->prodNext = NULL; 

    return copy;
}

// Simulate a purchase (assign products to a customer) [add to head of customer's cart]
void addToCart(Customer *customer, Product *item)
{
	if(customer == NULL || item == NULL)
	{
		fprintf(stderr, "Invalid customer or product.\n");
		return;
	}

	Product *copy = createProductCopy(item);
	if(copy == NULL)
	{
		fprintf(stderr, "Error in copying product.\n");
		return;
	}
	copy->prodNext = customer->cart; // add new product to head of cart
	customer->cart = copy;

	printf("%s added successfully to customer's cart.\n", item->name);
}

// Calculate total including discounts (entered as a percentage e.g. 20, 15, etc.) and tax (set previously)
float calculateTotal(Customer *customer, int discount)
{
	if(customer == NULL || customer->cart == NULL)
	{
		printf("Null passenger or an empty cart--no total to calculate.\n");
		return 0.0;
	}
	Product *list = customer->cart;
	float total = 0.0;
	int count = 0; // how many items in the cart
	while(list != NULL)
	{
		count++;
		total += list->price;
		list = list->prodNext;
	}

	if(count >= DISCOUNT_THRESHOLD)
	{
		printf("This customer has qualified for a discount after purchasing more than %d items.\n", DISCOUNT_THRESHOLD);
		total -= total*DISCOUNT_ITEMS; // 5% percent discount
	}

	if(discount != 0) {
		total -= total*(discount/100);
	}

	total += total*TAX_RATE;

	return total;
}

void printCustomers(Customer *head)
{
	Customer *curr = head;
	if(curr == NULL)
	{
		printf("Customer list is empty.\n");
		return;
	}

	int i = 1;
	printf("---Current Customers in System---\n");
	while(curr != NULL)
	{
		printf("Customer #%d\nName: %s\nAddress: %s\nEmail: %s\n\n", i++, curr->name, curr->address, curr->email);
		curr = curr->cusNext;
	}
}

void printCustomerNames(Customer *head)
{
	Customer *curr = head;
	if(curr == NULL)
	{
		printf("Customer list is empty.\n");
		return;
	}

	int i = 1;
	printf("---Current Customers in System---\n");
	while(curr != NULL)
	{
		printf("Customer #%d\nName: %s\n", i++, curr->name);
		curr = curr->cusNext;
	}
}

void printProducts(Product *head)
{
	Product *curr = head;
	if(curr == NULL)
	{
		printf("Product list is empty.\n");
		return;
	}
	
	int i = 1;
	printf("---Current Catalogue---\n");
	while(curr != NULL)
	{
		printf("%3d: %-60s %10.2feur\n", i++, curr->name, curr->price);
		curr = curr->prodNext;
	}
}

void printCart(Product *head)
{
	Product *curr = head;
	if(curr == NULL)
	{
		printf("Customer's cart is empty.\n");
		return;
	}
	
	int i = 1;
	printf("---Current Cart---\n");
	while(curr != NULL)
	{
		printf("%3d: %-60s %10.2feur\n", i++, curr->name, curr->price);
		curr = curr->prodNext;
	}
}

void clearCart(Customer *customer)
{
	if(customer == NULL || customer->cart == NULL)
	{
		return;
	}
	Product *curr = customer->cart;
	Product *next;

	while(curr != NULL)
	{
		next = curr->prodNext;
		free(curr->name);
		free(curr);
		curr = next;
	}

	customer->cart = NULL;
}

Customer * fillClientele(char *path)
{
	FILE *file = fopen(path, "r");
	if(!file)
	{
		fprintf(stderr, "Error: file used for loading client list not valid, check filename.");
		return NULL;
	}

	Customer *ret = NULL;

	char line[MAX_LINE];
	const char delimiters[] = ",";

	char name[NAME_MAX+1];
	char address[ADDR_MAX+1];
	char email[EMAIL_MAX+1];

	while(fgets(line, sizeof(line), file) != NULL)
	{
		line[strcspn(line, "\n")] = 0; // removes newline character if present
		const char *token = strtok(line, delimiters);
		if(token != NULL)
		{
			strncpy(name, token, NAME_MAX);
			name[NAME_MAX] = '\0';
			if((token = strtok(NULL, delimiters)) != NULL)
			{
				strncpy(address, token, ADDR_MAX);
				address[ADDR_MAX] = '\0';
			}
			else {
				fprintf(stderr,"Error reading address");
				fclose(file);
				return NULL;
			}
			if((token = strtok(NULL, delimiters)) != NULL)
			{
				strncpy(email, token, EMAIL_MAX);
				email[EMAIL_MAX] = '\0';
			}
			else {
				fprintf(stderr,"Error reading email");
				fclose(file);
				return NULL;
			}
		}
		else {
			fprintf(stderr, "Error reading name");
			fclose(file);
			return NULL;
		}
		Customer *newCus = createCustomer(name, address, email);
		if(newCus == NULL) {
			//fprintf(stderr, "Error allocating memory for new customer.\n");
			fclose(file);
			return NULL;
		}
		ret = addCustomer(&ret, newCus);
	}

	fclose(file);
	return ret;
}

// From file, loads all products onto linked list to manage
Product * fillCatalogue(char *path)
{
	FILE *file = fopen(path, "r");
	if(!file)
	{
		fprintf(stderr, "Error: file used for loading catalogue not valid, check filename.");
		return NULL;
	}

	Product *ret = NULL;

	char line[MAX_LINE];
	const char delimiters[] = ",";

	while(fgets(line, sizeof(line), file) != NULL)
	{
		char *productName; // will hold name (memory to be allocated with malloc)
		float price;	
		char *token = strtok(line, delimiters);
		if(token != NULL)
		{
			//printf("token: %s\n", token);
			//printf("strlen of token: %d\n", strlen(token));
			productName = (char *)malloc(strlen(token) + 1);
			if(productName != NULL)
			{
				strcpy(productName, token);
				//printf("product read: %s\n", productName);
			}
			else {
				fprintf(stderr, "Error: could not allocate space for product name when filling catalogue.");
				fclose(file);
				return NULL;
			}
			token = strtok(NULL, delimiters);
			if(token != NULL)
			{
				price = atof(token);
				//printf("price read: %.2f\n", price);
			}
			else {
				fprintf(stderr, "Error: could not assign price for product when filling catalogue.");
				free(productName);
				fclose(file);
				return NULL;
			}
		}
		else {
			fprintf(stderr, "Error reading first token in catalogue file.\n");
			fclose(file);
			return NULL;
		}
		Product *newProd = createProduct(productName, price);
		if(newProd == NULL)
		{
			//fprintf(stderr, "Error allocating space from new product node.\n");
			free(productName);
			fclose(file);
			return NULL;
		}
		//printf("newProd name: %s\t price: %.2f\n", newProd->name, newProd->price);
		ret = addProduct(&ret, newProd);
	}

	fclose(file);
	return ret;
}

// Changes were made, update files
void updateClientFile(char *path, Customer *head)
{
	FILE *clients = fopen(path, "w");
	if(!clients)
	{
		fprintf(stderr, "Error opening client file. Check filename.\n");
		return;
	}

	if(head == NULL) // all customers have been deleted
	{
		fprintf(clients, "\0");
	}

	Customer *curr = head;
	while(curr != NULL)
	{
		fprintf(clients, "%s,%s,%s\n", curr->name, curr->address, curr->email);
		curr = curr->cusNext;
	}

	fclose(clients);
}

void updateCatalogueFile(char *path, Product *head)
{
	FILE *products = fopen(path, "w");

	if(!products)
	{
		fprintf(stderr, "Error opening catalogue file. Check filename.\n");
		return;
	}

	if(head == NULL) // all products have been deleted
	{
		fprintf(products, "\0");
	}

	Product *curr = head;
	while(curr != NULL)
	{
		fprintf(products, "%s,%.2f\n", curr->name, curr->price); 
		curr = curr->prodNext;
	}

	fclose(products);
}

void printMenu(void)
{
	printf("---Main Menu---\n");
	printf("0. Exit\n1. Add a customer\n2. Edit customer details\n3. Delete customer information\n4. Add a product\n");
	printf("5. Edit product information\n6. Delete a product\n7. Simulate a purchase\n8. Display all customers\n");
	printf("9. Display current product list\n");
	printf("\nEnter your choice: ");
}

int main(int argc, char *argv[])
{
	if(argc < 2)
	{
		printf("No input files found, initiate first time execution? Warning: Files may be overwritten.\n");
		printf("Y/N: ");
		char in;
		scanf("%c", &in);
		if(in == 'Y' || in == 'y') {
			printf("Initializing product catalogue...\n");

			FILE *file = fopen("barcelona_catalogue.csv", "w");
			fprintf(file, "UCL Men's home jersey 24/25 FC Barcelona,114.99\n");
			fprintf(file, "UCL Men's home jersey 24/25 FC Barcelona - Player's Edition,164.99\n");
			fprintf(file, "Hoodie navy Barca 1899,64.99\n");
			fprintf(file, "Hoodie Barca Crest navy - Junior,44.99\n");
			fprintf(file, "FC Barcelona retro jacket,94.99\n");
			fprintf(file, "1995-97's Season Jersey,69.99\n");
			fprintf(file, "FC Barcelona 1991-92 Retro Away Jersey,69.99\n");
			fprintf(file, "FC Barcelona Captain's Jersey,69.99\n");
			fprintf(file, "Retro sweatshirt FC Barcelona,69.99\n");
			fprintf(file, "T-shirt black Barca,29.99\n");
			fclose(file);
			printf("Catalogue csv created under 'barcelona_catalogue.csv'.\n");

			printf("Initializing customer list...\n");
			FILE *file2 = fopen("customers.csv", "w");
			fprintf(file2, "John Egbert,123 Main Road,jegbert@gmail.com\n");
			fprintf(file2, "Paula Jameson,1502 North Apartment,paula2001@gmail.com\n");
			fprintf(file2, "Joy Goodsaints,22 Orchid Drive,goodsaintsjoy2@gmail.com\n");
			fclose(file2);
			printf("Customer csv file created under 'customers.csv'.\n");

			// calculate hash for file integrity check
			unsigned char hash[SHA256_DIGEST_LENGTH];

			if (generateFileHash("barcelona_catalogue.csv", hash) == 0) {
                writeHashToFile("barcelona_catalogue_hash.txt", hash, SHA256_DIGEST_LENGTH);
            }

            // Generate and store the hash for customers.csv
            if (generateFileHash("customers.csv", hash) == 0) {
                writeHashToFile("customers_hash.txt", hash, SHA256_DIGEST_LENGTH);
			}

			printf("Program closing, reopen with %s barcelona_catalogue.csv customers.csv\n", argv[0]);

			return EXIT_SUCCESS;
		}
		else if(in == 'N' || in == 'n') 
		{
			printf("Reopen with %s barcelona_catalogue.csv customers.csv\n", argv[0]);
			return 1;
		}
		else {
			printf("Input not recognized, program exiting.\n");;
			return 2;
		}
	}
	
	// Check integrity of files
	unsigned char generatedHash[SHA256_DIGEST_LENGTH];
	unsigned char storedHash[SHA256_DIGEST_LENGTH];

	// Verify catalogue csv file
	if (readHashFromFile("barcelona_catalogue_hash.txt", storedHash, SHA256_DIGEST_LENGTH) == 0 &&
		generateFileHash("barcelona_catalogue.csv", generatedHash) == 0 &&
		!compareHashes(generatedHash, storedHash, SHA256_DIGEST_LENGTH))
	{
		fprintf(stderr, "File integrity verification failed for barcelona_catalogue.csv.\n");
		return EXIT_FAILURE;
	}

	// Verify client csv file
	if (readHashFromFile("customers_hash.txt", storedHash, SHA256_DIGEST_LENGTH) == 0 &&
		generateFileHash("customers.csv", generatedHash) == 0 &&
		!compareHashes(generatedHash, storedHash, SHA256_DIGEST_LENGTH))
	{
		fprintf(stderr, "File integrity verification failed for customers.csv.\n");
		return EXIT_FAILURE;
	}

	printf("File integrity check passed!\n");


	// Initialize product and customer LL and fill from inputted files
	Product *catalogue = fillCatalogue(argv[1]);
	Customer *clientele = fillClientele(argv[2]);

	// Check if any changes are made
	bool cusEdits = false;
	bool prodEdits = false;

	// Open menu and await user input
	int input;

	while(1)
	{
		printMenu();
		scanf("%d", &input);
		clearInputBuffer();

		if(input == 0)
		{
			clearInputBuffer();
			break;
		}

		switch(input)
		{
			case 1: // add a customer
			{
				char name[NAME_MAX+1];
				char addr[ADDR_MAX+1];
				char email[EMAIL_MAX+1];

				printf("Input the customer's name:\n");
				fgets(name, NAME_MAX+1, stdin);
				name[strcspn(name, "\n")] = 0;

				printf("Input the customer's address:\n");
				fgets(addr, ADDR_MAX+1, stdin);
				addr[strcspn(addr, "\n")] = 0;

				printf("Input the customer's email address:\n");
				fgets(email, EMAIL_MAX+1, stdin);
				email[strcspn(email, "\n")] = 0;

				Customer *newCustomer = createCustomer(name, addr, email);

				if(newCustomer == NULL) {
					break;
				}

				clientele = addCustomer(&clientele, newCustomer);
				printf("%s successfully added.\n", name);
				cusEdits = true;
			}
			break;
			case 2: // edit customer
			{
				char name[NAME_MAX+1];
				char choice[20];
				char replace[ADDR_MAX+1]; // addr is the biggest possible field
				printCustomerNames(clientele);
				printf("Which customer's information do you want to edit? (Full name, case sensitive)\n");
				fgets(name, NAME_MAX+1, stdin);
				name[strcspn(name, "\n")] = 0;

				Customer *cus = findByName(clientele, name);
				if(cus == NULL)
				{
					printf("Invalid name, customer not found.\n");
					break;
				}
				
				printf("What field would you like to edit, Name, Address, or Email?\n");
				fgets(choice, 20, stdin);
				choice[strcspn(choice, "\n")] = 0;

				printf("Enter the replacement:\n");
				fgets(replace, ADDR_MAX+1, stdin);
				replace[strcspn(replace, "\n")] = 0;
				
				clientele = editCustomer(clientele, name, choice, replace);
				cusEdits = true;
			}
			break;
			case 3: // delete customer
			{
				char name[NAME_MAX+1];
				printCustomerNames(clientele);
				printf("Whose information do you want to delete? (Full name, case sensitive)\n");
				fgets(name, NAME_MAX+1, stdin);
				name[strcspn(name, "\n")] = 0;

				Customer *h = deleteCustomer(clientele, name);
				if(h == NULL)
				{ // deleteCustomer throws error message
					break;
				}
				clientele = h;
				printf("%s deleted from system.\n", name);
				cusEdits = true;
			}
			break;
			case 4: // add product
			{
				char name[MAX_LINE];
				float price;

				printf("Input name of product:\n");
				fgets(name, MAX_LINE+1, stdin);
				name[strcspn(name, "\n")] = 0;

				printf("Input its price:\n");
				scanf("%f", &price);

				clearInputBuffer();

				Product *newProduct = createProduct(name, price);
				if(newProduct == NULL)
				{
					break;
				}

				catalogue = addProduct(&catalogue, newProduct);
				printf("%s successfully added to catalogue.\n", name);
				prodEdits = true;
			}
			break;
			case 5: // edit product
			{
				int productID;
				char choice[20];
				
				printProducts(catalogue);
				printf("Which product's information do you want to edit? (Input its number): ");
				scanf("%d", &productID);
			
				clearInputBuffer();
			
				Product *target = productIDSearch(catalogue, productID);
				if(target == NULL)
				{
					break;
				}
				
				printf("\nWhat field would you like to edit, Name or Price?\n");
				fgets(choice, 20, stdin);
				choice[strcspn(choice, "\n")] = 0;
			
				if(strcmp(choice, "Name")==0 || strcmp(choice, "name")==0) 
				{
					char name[MAX_LINE];
					printf("Input replacement name:\n");
					fgets(name, MAX_LINE+1, stdin);
					name[strcspn(name, "\n")] = 0;
			
					char *newName = (char *)realloc(target->name, strlen(name) + 1);
			
					if(newName == NULL)
					{
						fprintf(stderr, "Reallocation for new product name failed.\n");
						break;
					}
					target->name = newName; // change pointer
					strcpy(target->name, name);
			
					printf("Product #%d has been renamed to %s.\n", productID, target->name);
				}
				else if(strcmp(choice, "Price")==0 || strcmp(choice, "price")==0)
				{
					float newPrice;
					scanf("%f", &newPrice);
			
					clearInputBuffer();
			
					target->price = newPrice;
					printf("Product #%d has had its price changed to %.2f.\n", productID, target->price);
				}
				else
				{
					printf("Invalid option inputted.\n");
					break;
				}
				prodEdits = true;
			}
			break;
			case 6: // delete product
			{
				int id;
				printProducts(catalogue);
				printf("Which product do you want to delete? Input their number: ");
				scanf("%d", &id);

				Product *target = productIDSearch(catalogue, id);
				if(target == NULL)
				{
					break;
				}

				deleteProduct(catalogue, target->name);
				prodEdits = true;
			}
			break;
			case 7: // simulate purchase
			{
				char name[NAME_MAX+1];
				int id;

				printCustomerNames(clientele);
				printf("Which customer is undergoing a purchase? (Full name, case sensitive)\n");
				fgets(name, NAME_MAX+1, stdin);
				name[strcspn(name, "\n")] = 0;

				//clearInputBuffer();

				Customer *customer = findByName(clientele, name);
				if(customer == NULL)
				{
					printf("Invalid customer name (case sensitive, full name necessary).\n");
					break;
				}

				printProducts(catalogue);
				printf("Input product number to add to customer's cart or 0 to finish transaction: ");

				while(scanf("%d", &id) == 1 && id != 0)
				{
					clearInputBuffer();

					Product *target = productIDSearch(catalogue, id);
					if(target == NULL)
					{
						printf("Please try again: ");
						continue;
					}
					else {
						addToCart(customer, target);
						printCart(customer->cart);
					}
					printProducts(catalogue);
					printf("Input product number to add to customer's cart or 0 to finish transaction: ");
				}

				clearInputBuffer();

				int discount;
				printf("\nInput any customer discount as a whole percentage (0, 15, 20, etc.):\n");
				if(scanf("%d", &discount) != 1) {
					printf("Invalid input for discount. No discount applied.\n");
					discount = 0;
				}
				clearInputBuffer();

				float total = calculateTotal(customer, discount);
				printf("After taxes, customer owes %.2f euros.\n", total);

				clearCart(customer); // clears for potential future purchases
			}
			break;
			case 8: // display customers
			{
				printCustomers(clientele);
			}
			break;
			case 9: // display product catalogue
			{
				printProducts(catalogue);
			}
			break;
			default: printf("Input a valid option (0 to 9).\n");
			break;
		}
	}

	// After user is done using the system, update the files
	clearInputBuffer();
	while(1)
	{
		if(cusEdits || prodEdits)
		{
			printf("Save changes to external files?\nY/N: ");
			char choice;
			scanf(" %c", &choice);

			if(choice == 'Y' || choice == 'y') {
				// check what was edited
				if(cusEdits && !prodEdits) // just clients changed
				{
					updateClientFile(argv[2], clientele);
					printf("\nCustomer file updated, program exiting...\n");

					// update hash
					unsigned char hash[SHA256_DIGEST_LENGTH];
                    if (generateFileHash(argv[2], hash) == 0) {
                        writeHashToFile("customers_hash.txt", hash, SHA256_DIGEST_LENGTH);
                    }

					break;
				}
				else if(prodEdits && !cusEdits) // only catalogue changes
				{
					updateCatalogueFile(argv[1], catalogue);
					printf("\nCatalogue file updated, program exiting...\n");

					// Update hash
                    unsigned char hash[SHA256_DIGEST_LENGTH];
                    if (generateFileHash(argv[1], hash) == 0) {
                        writeHashToFile("barcelona_catalogue_hash.txt", hash, SHA256_DIGEST_LENGTH);
                    }
					
					break;
				}
				else // changes to both
				{
					updateClientFile(argv[2], clientele);
					updateCatalogueFile(argv[1], catalogue);
					printf("\nCustomer and catalogue file updated, program exiting...\n");

					// Update hashes for both files
                    unsigned char hash[SHA256_DIGEST_LENGTH];
                    if (generateFileHash(argv[2], hash) == 0) {
                        writeHashToFile("customers_hash.txt", hash, SHA256_DIGEST_LENGTH);
                    }
                    if (generateFileHash(argv[1], hash) == 0) {
                        writeHashToFile("barcelona_catalogue_hash.txt", hash, SHA256_DIGEST_LENGTH);
					}

					break;
				}
			}
			if(choice == 'N' || choice == 'n') {
				printf("Changes discarded, program exiting...\n");
				break;
			}
			else {
				printf("Input not recognized. Try again: ");
			}
		}
		else // no changes made, no menu to be displayed
		{
			printf("\nProgram exiting...\n");
			break;
		}
	}
	
	return EXIT_SUCCESS;
}
