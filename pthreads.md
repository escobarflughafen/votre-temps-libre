# `pthread` (`std::thread`) in CPP


## Sample: Shared Message Queue

```c++
#include <iostream>
#include <vector>
#include <thread>
#include <string>
#include <mutex>
#include <queue>
#include <condition_variable>

// std::vector<std::string> shared_list = {};
std::queue<std::string> shared_list = {};
std::mutex mtx_list;
std::condition_variable cv_list;
bool ready = false;

void keyboard()
{
    bool quit = false;

    std::string input;

    while (!quit)
    {
        //std::cout << "\n" << "Enter a word: ";
        std::cin >> input;
        if (input == "!")
        {
            quit = true;
        }
        {
            std::lock_guard<std::mutex> lck(mtx_list);
            shared_list.push(input);
            ready = true;
            cv_list.notify_all();
        }
        input = "";
    }
    std::cout << "Exiting the keyboard thread." << std::endl;
}

void screen()
{
    bool quit = false;
    std::string to_output;

    while (!quit)
    {
        std::unique_lock<std::mutex> lck(mtx_list);
        while (!ready)
        {
            cv_list.wait(lck);
        }

        while (!shared_list.empty())
        {
            to_output = shared_list.front();
            if (to_output == "!")
            {
                quit = true;
                break;
            }

            std::cout << to_output << "\t";

            shared_list.pop();
        }
        std::cout << std::endl;
        ready = false;
    }

    std::cout << "Exiting the screen thread." << std::endl;
}

int main()
{
    std::thread t_keyboard(keyboard);
    std::thread t_screen(screen);

    std::cout << "Here is a demo of using std::thread to control a shared list." << std::endl;

    t_keyboard.join();
    t_screen.join();

    return 0;
}
```



## Components

- `vector`
  - 
- `queue`
  - member variables
  - member functions
- `thread`
  - member variables
  - member functions
- `mutex`
  - `std::mutex`
    - 
  - `std::lock_guard<>`
  - `std::unique_lock<>`
- `condition_variable`
  - `std::condition_variable`

