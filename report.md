cd your-project

git clone https://github.com/ne-kit-28/lab05
клонируем репозиторий

cd lab05
mkdir .github
cd .github
mkdir workflows
cd workflows
cat >> 123.yml

name: test build

on: [push]
jobs:
  build:
    
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name add gtest
      run: git clone https://github.com/google/googletest.git third-party/gtest -b release-1.11.0

    - name: Install lcov
      run: sudo apt-get install -y lcov

    - name: Config banking with tests
      run: cmake -H. -B ${{github.workspace}}/build -DBUILD_TESTS=ON

    - name: Build banking
      run: cmake --build ${{github.workspace}}/build

    - name: Run tests
      run: build/check

    - name: Do lcov stuff
      run: lcov -c -d build/CMakeFiles/banking.dir/banking/ --include *.cpp --output-file ./coverage/lcov.info

    - name: Publish to coveralls.io
      uses: coverallsapp/github-action@v1.1.2
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}


"ctrl+D"
cd ..
cd ..
cat >> CMakeLists.txt

cmake_minimum_required(VERSION 3.4)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
option(BUILD_TESTS "Build tests" OFF)
if(BUILD_TESTS)
  add_compile_options(--coverage)
endif()
project (banking)
add_library(banking STATIC ${CMAKE_CURRENT_SOURCE_DIR}/banking/Transaction.cpp ${CMAKE_CURRENT_SOURCE_DIR}/banking/Account.cpp)
target_include_directories(banking PUBLIC
${CMAKE_CURRENT_SOURCE_DIR}/banking )
target_link_libraries(banking gcov)
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB BANKING_TEST_SOURCES tests/*.cpp)
  add_executable(check ${BANKING_TEST_SOURCES})
  target_link_libraries(check banking gtest_main)
  add_test(NAME check COMMAND check)
endif()

"ctrl+D"
mkdir tests
cd tests
cat >> test_Account.cpp

include <Account.h>
#include <gtest/gtest.h>
TEST(Account, Banking){
	Account test(0,0);
	ASSERT_EQ(test.GetBalance(), 0);
	ASSERT_THROW(test.ChangeBalance(100), std::runtime_error);
	test.Lock();
	ASSERT_NO_THROW(test.ChangeBalance(100));
	ASSERT_EQ(test.GetBalance(), 100);
	ASSERT_THROW(test.Lock(), std::runtime_error);
	test.Unlock();
	ASSERT_THROW(test.ChangeBalance(100), std::runtime_error);
}

"ctrl+D"

cd ..
mkdir coverage
cd coverage
touch temp.txt
cd ..
git status
git add .
git commit -m "Lab has been planted"
cd tests
cat >> test_Transaction.cpp

#include <Account.h>
#include <Transaction.h>
#include <gtest/gtest.h>
TEST(Transaction, Banking){
//создаём константы. base_fee для полноценного теста должен быть хотя бы 51.
	const int base_A = 5000, base_B = 5000, base_fee = 100;
//создаём тестовые объекты
	Account Alice(0,base_A), Bob(1,base_B);
	Transaction test_tran;
//проверяем конструктор по умолчанию и сеттеры-геттеры
	ASSERT_EQ(test_tran.fee(), 1);
	test_tran.set_fee(base_fee);
	ASSERT_EQ(test_tran.fee(), base_fee);
//проверяем случаи когда транзакция не проходит
	ASSERT_THROW(test_tran.Make(Alice, Alice, 1000), std::logic_error);
	ASSERT_THROW(test_tran.Make(Alice, Bob, -50), std::invalid_argument);
	ASSERT_THROW(test_tran.Make(Alice, Bob, 50), std::logic_error);
	if (test_tran.fee()*2-1 >= 100)
		ASSERT_EQ(test_tran.Make(Alice, Bob, test_tran.fee()*2-1), false);
//проверяем, что всё правильно лочится
	Alice.Lock();
	ASSERT_THROW(test_tran.Make(Alice, Bob, 1000), std::runtime_error);
	Alice.Unlock();
//проверяем что если входные параметры правильные, то транзакция проходит православно
	ASSERT_EQ(test_tran.Make(Alice, Bob, 1000), true);
	ASSERT_EQ(Bob.GetBalance(), base_B+1000);
	ASSERT_EQ(Alice.GetBalance(), base_A-1000-base_fee);
//проверяем что транзакция не проходит, если не хватает средств
	ASSERT_EQ(test_tran.Make(Alice, Bob, 3900), false);
	ASSERT_EQ(Bob.GetBalance(), base_B+1000);
	ASSERT_EQ(Alice.GetBalance(), base_A-1000-base_fee);
}

"ctrl+D"

