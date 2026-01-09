# Animal Adoption
This assignment guides you through building an animal adoption system with user login functionality, using POJOs, enums, file I/O, and composition to model animals and pets while practicing structured Java project setup and user interaction.
import java.util.ArrayList;
import java.time.LocalDate;

public class AnimalAdoptionSystem {
    private BFF helper;
    private ArrayList<User> userDatabase;
    private User currentUser;
    private ArrayList<AnimalType> allAnimals;

    public AnimalAdoptionSystem(BFF helper, ArrayList<User> userDatabase, User currentUser, ArrayList<AnimalType> allAnimals) {
        this.helper = helper;
        this.userDatabase = userDatabase;
        this.currentUser = currentUser;
        this.allAnimals = allAnimals;
    }

    public AnimalAdoptionSystem() {
        this.helper = new BFF();
        this.userDatabase = new ArrayList<>();
        this.currentUser = null;
        this.allAnimals = Animal_FileReader.readEmojiAnimalFile();
        
    }

    public void run() {
        boolean quit = false;
        while (!quit) {
            System.out.println(LoginMenu.getMenuString());
            // helper to choose option
            int userChoice = helper.inputInt(">", 0, LoginMenu.getMaxChoice());
            LoginMenu option = LoginMenu.getOptionNumber(userChoice);
            switch (option) {
                case MAKE_FAKE_ACCOUNTS:
                    makeFakeAccounts();
                    break;
                case DISPLAY_USER_NAMES:
                    displayAccounts();
                    break;
                case CREATE_ACCOUNT:
                    createAccount();
                    break;
                case LOGIN:
                    login();
                    break;
                case LOGOUT:
                    logOut();
                    break;
                case CHANGE_PASSWORD:
                    changePassword();
                    break;
                case ADOPT_PET:
                    adoptPet();
                    break;
                case DISPLAY_USER:
                    displayUser();
                    break;
                case QUIT:
                    quit = true;
                    System.out.println("Goodbye:)");
                    break;
            }
        }
    }

    public void makeFakeAccounts() {
        // create Pet pets
        Pet petA1 = new Pet(allAnimals.get(9), "Skippy");
        Pet petA2 = new Pet(allAnimals.get(10), "Jack");
        Pet petA3 = new Pet(allAnimals.get(11), "Peanut");
        Pet petA4 = new Pet(allAnimals.get(12), "Jelly");
        Pet petA5 = new Pet(allAnimals.get(13), "Pikachu");


        // create pet arrays
        Pet[] petArray1 = new Pet[]{petA1, petA2, null};
        Pet[] petArray2 = new Pet[]{petA2, null, petA3};
        Pet[] petArray3 = new Pet[]{petA4, petA5};
        Pet[] petArray4 = new Pet[]{petA5, petA1, null, null};

        // Users
        User user1 = new User("Kendra", "pass", LocalDate.parse("2000-05-04"), petArray1);
        User user2 = new User("Ken", "pass", LocalDate.parse("2000-05-05"), petArray2);
        User user3 = new User("Henry", "pass", LocalDate.parse("2000-05-06"), petArray3);
        User user4 = new User("Kelsey", "pass", LocalDate.parse("2000-05-07"), petArray4);
        // add them to the database
        userDatabase.add(user1);
        userDatabase.add(user2);
        userDatabase.add(user3);
        userDatabase.add(user4);

    }

    public void displayAccounts() {
        System.out.println("All users: ");
        for (User display : userDatabase) {
            System.out.println(display.getUserName());
        }
    }
    public void createAccount() {
        String usersName = helper.input("User's name? ");
        while (findUser(usersName) != null) {
            System.out.println("Sorry, there's already a user with that name. ");
            usersName = helper.input("Try a different username: ");
        }
        System.out.println("Birthday?");
        int year = helper.inputInt("Year: ");
        int month = helper.inputInt("Month: ");
        int day = helper.inputInt("Day: ");
        String password = helper.input("Password: ");
        String confirmPassword = helper.input("Confirm password: ");
        while (!password.equals(confirmPassword)) {
            System.out.println("Account not created. Passwords didn't match");
            confirmPassword = helper.input("Confirm password: ");
        }
        // String userName, String password, LocalDate bday, Pet[] pets
        userDatabase.add(new User(usersName, password, year, month, day));
        System.out.println("Created a new account and logged in user");
    }

    public void login() {
        String name = helper.input("Enter username: ");
        User inputName = findUser(name);
        if (inputName == null) {
            System.out.println("No user found with name: " + name);
        }
        String pass = helper.input("Enter password: ");
        if (!inputName.getPassword().equals(pass)) {
            System.out.println("Password doesn't match");
        }
        currentUser = inputName;
        System.out.println(name + " is now logged in.");
    }

    public void logOut() {
        currentUser = null;
    }

    public void changePassword() {
        if (currentUser != null){
            String pass = helper.input("Enter your old password: ");
            if (currentUser.getPassword().equalsIgnoreCase(pass)) {
                String newPass = helper.input("Enter your new password: ");
                currentUser.updatePassword(pass, newPass);
            }
        }
    }

    public void adoptPet() {
        if (currentUser != null) {
            System.out.println("You have " + allAnimals.size() + " animals to choose from");
            for (int i = 0; i < AnimalCategory.values().length; i++) {
                System.out.println(AnimalCategory.values()[i]);
            }
            int choice = helper.inputInt("Please enter the category by number: ", 0, 5);
            AnimalCategory animal = AnimalCategory.values()[choice];
            ArrayList<AnimalType> type;
            ArrayList<AnimalType> subList = specificList(animal, allAnimals);
                // get sublist of category
                // list of different animals
                // get user input of what they want
            System.out.println("There are " + subList.size() + " animals to choose from in " + animal + " category");
            boolean narrow = helper.inputYesNo("Would you like to narrow that to ones that make good pets?");
            if(narrow){
                type = specificList(true, subList);
            }
            else {
                boolean narrow2 = helper.inputYesNo("Are you are looking for something exotic that isn't typically a pet");
                if(narrow2){
                    type = specificList(false, subList);
                }
                else{
                    type = subList;
                }
            }
                for (int i = 0; i < type.size(); i++) {
                    System.out.println(type.get(i));
                }
        }
    }

        // print out options to user, select and name animal
    

    public ArrayList<AnimalType> specificList(boolean isPet, ArrayList<AnimalType> type) {
        ArrayList<AnimalType> x = new ArrayList<>();

        for (AnimalType y: type) {
            if (y.isPet() == isPet) {
                x.add(y);
            }
        }
        return x;
    }

    public ArrayList<AnimalType> specificList(AnimalCategory category, ArrayList<AnimalType> type) {
        ArrayList<AnimalType> x = new ArrayList<>();
        for (AnimalType y: type) {
            if (y.getCategory() == (category)) {
                x.add(y);
            }
        }
        return x;
    }

    public void displayUser() {
        // make an empty null user object, then have a while loop while user == null, ask for a username using helper method, use find user method to look for that name and update user variable accordingly
        User u1 = null;
        while (u1 == null) {
            String userName = helper.input("Please enter the username you are looking for: ");
            u1 = findUser(userName);
        }
    }

    private User findUser(String name) {
        boolean found = false;
        User u = null;
        int i = 0;
        while (!found && i < userDatabase.size()) {
            User data = userDatabase.get(i);
            if (data.getUserName().equalsIgnoreCase(name)) {
                found = true;
                u = data;
            }
            i++;
        return u;

        }
        for (User hi : userDatabase) {
            while (hi.getUserName().equals(name)) {
                return hi;
            }
        }
        return null;
    }

    private boolean verifyUser() {
        if (currentUser == null) {
            System.out.println("Someone needs to be logged in in order to complete this action");
            return false;
        }
        return true;
    }

    public static void main(String[] args) {
        AnimalAdoptionSystem fb = new AnimalAdoptionSystem();
        fb.run();
    }

}
