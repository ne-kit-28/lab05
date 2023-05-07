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

