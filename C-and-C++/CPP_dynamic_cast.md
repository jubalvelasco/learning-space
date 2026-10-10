# Down casting & `dynamic_cast`
    
- **Upcasting** - assigning a pointer of a ```derived class``` to a pointer of a ```base class``` (done implicitly by the compiler and always safe, so there are no problems with this)
- **Down casting** - the act of assigning a pointer of a ```base class``` to a pointer of a ```derived class```, here we should use dynamic_cast for safe casting

**A sample program showing how to use a dynamic cast**
    
NOTE: dynamic_cast (for downcasting) can only be used with polymorphic classes, i.e. classes with at least one virtual function. If the cast fails, it returns `nullptr` for pointers, or throws `std::bad_cast` for references.
    
```c++
#include <iostream>
#include <string>

class person
{
    private:
        std::string name;
        unsigned int age;
        char gender;
    
    public:
        person(const std::string& ID, const std::string& name, unsigned int age, char gender)
        : name(name), age(age), gender(gender), ID(ID) {}

        std::string ID;

        std::string getName() { return name; }
        virtual ~person() { /* Note: Destructors should always have a definition */ }
};

class student : public person
{
    public:
        student(const std::string& ID, const std::string& name, unsigned int age, char gender)
        : person(ID,name,age,gender), Student_ID("0xFFHG"+ID) {}

        std::string Student_ID;

        ~student() { /* an empty definition */ }
};

class teacher : public person
{
    public:
        teacher(const std::string& ID, const std::string& name, unsigned int age, char gender)
        : person(ID,name,age,gender), Teacher_ID("0xFAHJ"+ID) {}

        std::string Teacher_ID;

        ~teacher() { /* an empty definition */ }
};

int main()
{
    person* Micha = new student("0232-1232","Micha", 21, 'F');
    person* Aldino = new teacher("2445-8270","Aldino", 33, 'M');

    student* student1 = dynamic_cast<student*>(Micha);
    student* student2 = dynamic_cast<student*>(Aldino);

    // ---------- check if student 1 is a real student ----------
    if(student1!=nullptr)
        std::cout << student1->getName() << " is a real student\n";
    else
        std::cout << "Detected a fake student\n";
    
    // ---------- check if student 2 is a real student ----------
    if(student2!=nullptr)
        std::cout << student2->getName() << " is a real student\n";
    else
        std::cout << "Detected a fake student\n";
    
    delete Micha;
    delete Aldino;
    return 0;
}
```
    
**Output if dynamic_cast is used**
```shell
Micha is a real student
Detected a fake student
```

**Output if static_cast is used**
```shell
Micha is a real student
Aldino is a real student
```

NOTE: with `static_cast` there is no runtime check, so `student2` points to a `teacher` object treated as a `student`. Using it is **undefined behavior**; the output above is just what typically happens, it is not guaranteed.
