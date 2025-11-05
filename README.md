#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_ACCOUNTS 100

typedef struct {
    int acc_no;
    char name[50];
    char acc_type;        // 'S' for savings, 'C' for current
    double balance;
    int is_active;        // 1 = active, 0 = deleted/inactive
} Account;

/* Global array of accounts (simple approach) */
Account accounts[MAX_ACCOUNTS];
int next_acc_no = 1001;   // starting account number
int total_accounts = 0;

/* Function prototypes */
int addAccount();
int findAccountIndexByNo(int acc_no);
void displayAccount(const Account *acc);
void listAccounts();
void depositWithdraw(int acc_no, int is_deposit);
void modifyAccount(int acc_no);
void deleteAccount(int acc_no);
void pause_for_enter();

/* Utility to read a line of input safely into buffer */
void read_line(char *buffer, int size) {
    if (fgets(buffer, size, stdin) != NULL) {
        /* Remove trailing newline if present */
        size_t len = strlen(buffer);
        if (len > 0 && buffer[len-1] == '\n') buffer[len-1] = '\0';
    }
}

/* Add a new account */
int addAccount() {
    if (total_accounts >= MAX_ACCOUNTS) {
        printf("Maximum accounts reached (%d). Cannot add more.\n", MAX_ACCOUNTS);
        return -1;
    }

    Account new_acc;
    new_acc.acc_no = next_acc_no++;

    printf("Enter account holder name: ");
    read_line(new_acc.name, sizeof(new_acc.name));
    if (strlen(new_acc.name) == 0) {
        printf("Name cannot be empty. Account not created.\n");
        return -1;
    }

    char type_input[10];
    while (1) {
        printf("Enter account type (S for Savings / C for Current): ");
        read_line(type_input, sizeof(type_input));
        if (strlen(type_input) == 0) continue; // show how continue can be used
        if (type_input[0] == 'S' || type_input[0] == 's') {
            new_acc.acc_type = 'S';
            break;
        } else if (type_input[0] == 'C' || type_input[0] == 'c') {
            new_acc.acc_type = 'C';
            break;
        } else {
            printf("Invalid type. Try again.\n");
            continue;
        }
    }

    double init_balance = 0.0;
    while (1) {
        char bal_input[50];
        printf("Enter initial deposit (>= 0): ");
        read_line(bal_input, sizeof(bal_input));
        if (strlen(bal_input) == 0) { printf("Please enter a number.\n"); continue; }
        if (sscanf(bal_input, "%lf", &init_balance) != 1) {
            printf("Invalid number. Try again.\n");
            continue;
        }
        if (init_balance < 0) { printf("Balance cannot be negative.\n"); continue; }
        break;
    }
    new_acc.balance = init_balance;
    new_acc.is_active = 1;

    /* Store in global array */
    accounts[total_accounts++] = new_acc;
    printf("Account created successfully. Account Number: %d\n", new_acc.acc_no);
    return new_acc.acc_no;
}

/* Find index by account number. Returns -1 if not found or inactive. */
int findAccountIndexByNo(int acc_no) {
    for (int i = 0; i < total_accounts; ++i) {
        if (accounts[i].acc_no == acc_no && accounts[i].is_active) {
            return i;
        }
    }
    return -1;
}

/* Display a single account (pointer used to show pointer usage) */
void displayAccount(const Account *acc) {
    if (acc == NULL) return;
    printf("Account No : %d\n", acc->acc_no);
    printf("Name       : %s\n", acc->name);
    printf("Type       : %c\n", acc->acc_type);
    printf("Balance    : %.2f\n", acc->balance);
}

/* List all active accounts using pointer arithmetic and continue */
void listAccounts() {
    if (total_accounts == 0) {
        printf("No accounts available.\n");
        return;
    }

    printf("Active Accounts:\n");
    printf("----------------\n");
    int any = 0;
    for (int i = 0; i < total_accounts; ++i) {
        Account *p = &accounts[i]; // pointer to current entry
        if (!p->is_active) {
            continue; // skip inactive entries (example use of continue)
        }
        any = 1;
        displayAccount(p);
        printf("----------------\n");
    }
    if (!any) printf("No active accounts found.\n");
}

/* Deposit or Withdraw */
void depositWithdraw(int acc_no, int is_deposit) {
    int idx = findAccountIndexByNo(acc_no);
    if (idx == -1) {
        printf("Account number %d not found.\n", acc_no);
        return;
    }
    Account *p = &accounts[idx];
    double amount = 0.0;
    char input[50];
    printf("Enter amount: ");
    read_line(input, sizeof(input));
    if (sscanf(input, "%lf", &amount) != 1 || amount <= 0) {
        printf("Invalid amount.\n");
        return;
    }

    if (is_deposit) {
        p->balance += amount;
        printf("Deposit successful. New balance: %.2f\n", p->balance);
    } else {
        if (amount > p->balance) {
            printf("Insufficient balance. Transaction cancelled.\n");
        } else {
            p->balance -= amount;
            printf("Withdrawal successful. New balance: %.2f\n", p->balance);
        }
    }
}

/* Modify account holder name and/or account type */
void modifyAccount(int acc_no) {
    int idx = findAccountIndexByNo(acc_no);
    if (idx == -1) {
        printf("Account number %d not found.\n", acc_no);
        return;
    }
    Account *p = &accounts[idx];
    printf("Current details:\n");
    displayAccount(p);

    char choice[10];
    printf("Modify name? (y/n): ");
    read_line(choice, sizeof(choice));
    if (choice[0] == 'y' || choice[0] == 'Y') {
        printf("Enter new name: ");
        read_line(p->name, sizeof(p->name));
    }

    printf("Modify account type? (y/n): ");
    read_line(choice, sizeof(choice));
    if (choice[0] == 'y' || choice[0] == 'Y') {
        char type_input[10];
        while (1) {
            printf("Enter new type (S/C): ");
            read_line(type_input, sizeof(type_input));
            if (strlen(type_input) == 0) { printf("Type cannot be empty.\n"); continue; }
            if (type_input[0] == 'S' || type_input[0] == 's') {
                p->acc_type = 'S';
                break;
            } else if (type_input[0] == 'C' || type_input[0] == 'c') {
                p->acc_type = 'C';
                break;
            } else {
                printf("Invalid type. Try again.\n");
                continue;
            }
        }
    }

    printf("Account modified successfully.\n");
    displayAccount(p);
}

/* "Delete" account by marking inactive */
void deleteAccount(int acc_no) {
    int idx = findAccountIndexByNo(acc_no);
    if (idx == -1) {
        printf("Account number %d not found.\n", acc_no);
        return;
    }
    accounts[idx].is_active = 0;
    printf("Account %d has been deleted (marked inactive).\n", acc_no);
}

/* Small utility to pause until user presses enter */
void pause_for_enter() {
    printf("Press Enter to continue...");
    while (getchar() != '\n');
}

/* MAIN: interactive menu */
int main() {
    char choice[10];
    printf("=== Simple Bank Management System ===\n");

    while (1) {
        printf("\nMain Menu:\n");
        printf("1. Create new account\n");
        printf("2. Display account by account number\n");
        printf("3. Deposit money\n");
        printf("4. Withdraw money\n");
        printf("5. Modify account\n");
        printf("6. Delete account\n");
        printf("7. List all accounts\n");
        printf("8. Exit\n");
        printf("Choose an option (1-8): ");

        read_line(choice, sizeof(choice));
        if (strlen(choice) == 0) { printf("Please enter a choice.\n"); continue; }

        int opt = atoi(choice);

        if (opt == 1) {
            addAccount();
        } else if (opt == 2) {
            char tmp[20];
            printf("Enter account number: ");
            read_line(tmp, sizeof(tmp));
            int acc_no = atoi(tmp);
            int idx = findAccountIndexByNo(acc_no);
            if (idx == -1) {
                printf("Account not found.\n");
            } else {
                displayAccount(&accounts[idx]);
            }
        } else if (opt == 3) {
            char tmp[20];
            printf("Enter account number for deposit: ");
            read_line(tmp, sizeof(tmp));
            int acc_no = atoi(tmp);
            depositWithdraw(acc_no, 1);
        } else if (opt == 4) {
            char tmp[20];
            printf("Enter account number for withdrawal: ");
            read_line(tmp, sizeof(tmp));
            int acc_no = atoi(tmp);
            depositWithdraw(acc_no, 0);
        } else if (opt == 5) {
            char tmp[20];
            printf("Enter account number to modify: ");
            read_line(tmp, sizeof(tmp));
            int acc_no = atoi(tmp);
            modifyAccount(acc_no);
        } else if (opt == 6) {
            char tmp[20];
            printf("Enter account number to delete: ");
            read_line(tmp, sizeof(tmp));
            int acc_no = atoi(tmp);
            deleteAccount(acc_no);
        } else if (opt == 7) {
            listAccounts();
        } else if (opt == 8) {
            printf("Exiting program. Goodbye!\n");
            break;
        } else {
            printf("Invalid option. Please choose 1-8.\n");
        }

        /* Small pause then loop again */
        printf("\n");
        printf("--------------\n");
    }

    return 0;
}
