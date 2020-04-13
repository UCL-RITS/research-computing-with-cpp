---
title: ITK
---

## Using ITK

### Introduction

* [Insight Segmentation and Registration Toolkit][ITK]
* Insight Journal for library additions
* Large community in medical image processing
* Deliberately no visualisation, see [VTK][VTK].


### C++ Principles

* Heavy use of Generic Programming
* Use of Template Meta-Programming
* Often perceived by "scientific programmers" (Matlab) as difficult
* Aim: demonstrate here, that we can now use it!
* Of particular interest
    * typedefs - make life easier
    * SmartPointers - reduce leaking memory
    * Iterators - fast image access
    * Object Factories - extensibility


### Architecture Concept

* Use of pipeline of filters
* Simple to plug image processing filters together
* Sometimes difficult to manage memory for huge images


### Filter Usage - 1

We work through a simple filter program. First, typedefs are aliases.

{% idio cpp/ITK %}

{% fragment typedefs, ITKAdd.cc %}


### Filter Usage - 2

Objects are constructed:

{% fragment construction, ITKAdd.cc %}


### Filter Usage - 3

Pipeline is executed:

{% fragment pipeline, ITKAdd.cc %}

More information on ITK Pipeline can be found in the
[ITK Software Guide][ITKSoftwareGuide].


### Smart Pointer Intro

Lets look at some interesting features.

* Smart Pointer
    * Class, like a pointer, but 'Smarter' (clever)
    * Typically, once allocated will automatically destroy the pointed to object
    * Implementations vary, STL, ITK, VTK, Qt, so read the docs
* So, in each class e.g. itkAddImageFilter

``` cpp
typedef AddImageFilter     Self
typedef SmartPointer<Self> Pointer
```

and so, its used like

``` cpp
ClassName::Pointer variableName = ClassName::New();
```


### Smart Pointer Class

In the SmartPointer itself

``` cpp
  /** Constructor to pointer p  */
  SmartPointer (ObjectType *p):
    m_Pointer(p)
  { this->Register(); }

  /** Destructor  */
  ~SmartPointer ()
  {
    this->UnRegister();
    m_Pointer = ITK_SP_NULLPTR;
  }

```

and

``` cpp
private:
  /** The pointer to the object referred to by this smart pointer. */
  ObjectType *m_Pointer;

  void Register()
  {
    if ( m_Pointer ) { m_Pointer->Register(); }
  }
```


### General Smart Pointer Usage

* Avoid use of explicit pairs of ```new/delete```
* Immediately assign object to SmartPointer
* Consistently (i.e. always) use SmartPointer
    * Pass (reference to) SmartPointer to function.
    * Can (but should you?) return SmartPointer from function.
    * Don't use raw pointer, and don't store raw pointers to objects.
    * You can't test raw pointer to check if object still exists.
* Object is deleted when last SmartPointer reference goes out of scope


### ITK SmartPointer

* ITK keeps reference count in itk::LightObject base class
* So, it can only be used by sub-classes of itk::LightObject
* Reference is held in the object
* Same method used in MITK, as MITK uses ITK concepts
* [VTK][VTKSmartPointers] has a SmartPointer that requires calling Delete explicitly (!!)
* STL has much clearer definition of different types of smart pointer
* Read [THIS][SmartPointerTutorial] tutorial


### Implementing a Filter

* ITK provides many image processing filters.
* But you can write your own easily
    * Single Threaded - override GenerateData()
    * Multi-Threaded - override ThreadedGenerateData()
* Now we see an example - thresholding, as we want to study the C++ not the image processing.


### Filter Impl - 1

Basic filter:

{% fragment intro, ITKThreshold.cc %}


### Filter Impl - 2

Boilerplate nested typedefs :

{% fragment boilerplate, ITKThreshold.cc %}


### Filter Impl - 3

Look at ITK Macros :

{% fragment macro, ITKThreshold.cc %}


### Filter Impl - 4

The main method :

{% fragment method, ITKThreshold.cc %}

{% endidio %}

### Iterators

* ITK provides many iterators
* Generic Programming means:
    * Suitable for n-dimensions
    * Suitable for all types of data
* Also, different image access concepts
    * Region of Interest
    * Random subsampling
    * No change in code
* So iterators enable you to traverse image and encapsulate the traversal mechanism in an iterator    
* Similar concept to STL ```.begin()```, ```.end()```
* See [ITK Software Guide][ITKSoftwareGuide]


### Private Constructors?

If you look at an ITK filter, you may notice for example

``` cpp
    protected:
      AddImageFilter() {}
      virtual ~AddImageFilter() {}

    private:
      AddImageFilter(const Self &);
      void operator=(const Self &);
```

* Copy constructor and copy assignment are private and not implemented
* Constructor and Destructor private. So how do you use?


### Static New Method

You will then see

``` cpp
  /** Method for creation through the object factory. */
  itkNewMacro(Self);
```

which if you hunt for long enough, you find this snippet

``` cpp
#define itkSimpleNewMacro(x)                                   
  static Pointer New(void)                                     
    {
    Pointer smartPtr = ::itk::ObjectFactory< x >::Create();
    if ( smartPtr.GetPointer() == ITK_NULLPTR )
      {
      smartPtr = new x;
      }
    return smartPtr;
    }
```
So, either this ```ObjectFactory``` creates it, or a standard ```new``` call.


### ObjectFactory::Create

In ```itk::ObjectFactory``` we ask factory to CreateInstance using a ```char*```

``` cpp
  static typename T::Pointer Create()
  {
    LightObject::Pointer ret = CreateInstance( typeid( T ).name() );
    return dynamic_cast< T * >( ret.GetPointer() );
  }
```

```CreateInstance``` works with either a base class name, or a class name
to return either a specific class, or a family of classes derived from a common base class.


### Why Object Factories?

* Rather than create objects directly
* Ask a class (ObjectFactory) to do it
* This class contain complex logic, not just a new operator
* So, we can
    * dynamically load libraries from ITK_AUTOLOAD_PATH at runtime
    * Have a list/map of current classes, and provide overrides
    * i.e swap in a GPU version instead of CPU
* More dynamic variant of FactoryMethod, AbstractFactory (See [GoF][GoF])


### File IO Example

In ```itkImageFileReader.hxx```

``` cpp
      std::list< LightObject::Pointer > allobjects =
        ObjectFactoryBase::CreateAllInstance("itkImageIOBase");
```

* We ask the factory for every class that is a sub-class of itkImageIOBase.
* Then we can ask each ImageIOBase sub-class if it can read a specific format.
* First one to reply true reads the image.
* In general case, ask ObjectFactoryBase for any class.


### Object Factory List of Factories

In class ObjectFactoryBase

``` cpp
class  ObjectFactoryBase:public Object
{
public:
  static std::list< ObjectFactoryBase * > GetRegisteredFactories();
```

This class maintains a static vector of ObjectFactoryBase. These
are added programmatically, via static initialisation or
dynamically via the ITK_AUTO_LOAD_PATH.


### PNG IO Factory

* Given ```itkPNGImageIO.h/cxx``` can read PNG images
* We see in ```itkPNGImageIOFactory.cxx```

``` cpp
PNGImageIOFactory::PNGImageIOFactory()
{
  this->RegisterOverride( "itkImageIOBase",
                          "itkPNGImageIO",
                          "PNG Image IO",
                          1,
                          CreateObjectFunction< PNGImageIO >::New() );
}
```
So, PNG factory says it implements a type of itkImageIOBase,
will return an itkPNGImageIO, and instantiates a function object that calls the right constructor.


### ObjectFactory Summary

* ObjectFactory defines a static vector of ObjectFactory
* ObjectFactory objects loaded:
    * Directly named in code at compile time
    * Via static initialisers when a dynamic library is loaded
    * Or from ITK_AUTOLOAD_PATH
* ObjectFactory returns one/all classes that implement a given class
* Static New method now asks factory for a class.
* So, you can override any ITK class.
* Why is above example not an infinite loop?




### ITK Summary

* Pipeline architecture for most filters
* Also includes a registration framework (see [ITK Software Guide][ITKSoftwareGuide])
* Smart Pointers - reference counting, automatic deletion
* Static New method with ObjectFactory to enable overriding any class at runtime
* Dynamic loading via ITK_AUTOLOAD_PATH
* Pipeline architecture - easy to prototype, once you know C++
* Write your own filter, unit test, generalise to n-dimension, of n-vectors.
* Easy to extend to multi-threading

[ITK]: http://www.itk.org
[VTK]: http://www.vtk.org
[SimpleITK]: http://www.simpleitk.org/
[ITKSoftwareGuide]: http://www.itk.org/ItkSoftwareGuide.pdf
[VTKSmartPointers]: http://www.vtk.org
[SmartPointerTutorial]: http://www.umich.edu/~eecs381/handouts/C++11_smart_ptrs.pdf
[GoF]: http://en.wikipedia.org/wiki/Design_Patterns
[ITKIO]: http://www.itk.org/Wiki/Plugin_IO_mechanisms
