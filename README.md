# std-enable_shared_from_this-and-constructors
Addresses the problem of using `std::enable_shared_from_this` with constructors

The motivation for this code is issues with `std::shared_ptr` and
`std::enable_shared_from_this`.

Consider code like this:
  `std::shared<SomeClass> ptr = std::make_shared<SomeClass>();`

Assume that `SomeClass`’s constructor uses `shared_from_this`. In this case, since
`SomeClass` is not yet owned by a `std::shared_ptr`, the `shared_from_this` call will
not work as expected. There seems to no way around this that I can find.
I wasn’t aware of this issue. It was a real kick in the pants.

The solution implemented here is two-stage construction. First the object is
constructed and assigned to a `std::shared_ptr`; then, if present, a
`post_construct` method is called with the same arguments. All code that uses
`shared_from_this` in the constructors should be moved to a `post_construct` method.
The intent of the "if present" is to enable incremental migration to two-stage
construction on demand.

Another issue addressed is with `std::make_shared`. This function requires that
the constructor be public. If we’re making factory methods for creation, we may
want to make the constructors non-public to stop misuse. In this case the code
uses `std::shared_ptr’s constructor` and a `new` call. The function that calls new
can be made a `friend` of the class to grant it access. We lose the advantages of
using `std::make_shared` but gain protection against incorrect object
instantiation.
