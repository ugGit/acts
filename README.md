# Compile Acts with nvc++

## Requirements

The issue might also be reproduced with older versions, but we went using these so far.

* gcc/11.2.0
* nvhpc/22.3    
* boost/1.72.0  
* eigen/3.4.0

## Configuration
To run the nvc++ compiler with the desired gcc version, a localrc file must be generated. This can be done by executing a script provided by nvhpc:

```
makelocalrc -gcc PATH_TO_GCC -gpp PATH_TO_G++ -x -d PATH_TO_LOCALRC_DIR
```

Then, the auxiliary script `nvc++_p` must be updated such that the variable `LOCALRC` points to PATH_TO_LOCALRC_DIR defined in the above command. E.g.:
```
LOCALRC="/home/nwachuch/bld6/nvcpp-tests/actscore-issue-recreation/localrc_gcc112"
```

Side note: the auxiliary script does remove some flags (e.g. `-Wno-unused-local-typedefs`) that CMake adds to the compilation process which are invalid for nvc++. Further, it is used to dynamically switch the compiler from gcc to nvc++ depending on the current target/library that is getting compiled.

## Compilation
The current state should allow compiling, linking, and running the program successfully. Therefore, pass the auxiliary script `nvc++_p` as compiler during the configuration of CMake as follows:
```
cmake -S . -B build -DCMAKE_CXX_COMPILER=$PWD/nvc++_p
```

Then build the program:
```
cmake --build build/ 
```

Which results in a lot of warnings and the following error:
```
"/home/nwachuch/bld6/acts/Core/include/Acts/Utilities/detail/MPL/type_collector.hpp", line 58: error: expression must have a constant value
        hana::filter(t_, [&](auto t) { return predicate(t); });
        ^
"/opt/boost/1.72.0/include/boost/hana/detail/ebo.hpp", line 62: note: attempt to access run-time storage
              : V(static_cast<T&&>(t))
                  ^
"/opt/boost/1.72.0/include/boost/hana/basic_tuple.hpp", line 69: note: called from:
              explicit constexpr basic_tuple_impl(Yn&& ...yn)
                                 ^
"/opt/boost/1.72.0/include/boost/hana/basic_tuple.hpp", line 96: note: called from:
          explicit constexpr basic_tuple(Yn&& ...yn)
                             ^
"/opt/boost/1.72.0/include/boost/hana/tuple.hpp", line 127: note: called from:
          constexpr tuple(Yn&& ...yn)
                    ^
"/opt/boost/1.72.0/include/boost/hana/tuple.hpp", line 316: note: called from:
          { return {static_cast<Xs&&>(xs)...}; }
            ^
"/opt/boost/1.72.0/include/boost/hana/fwd/core/make.hpp", line 61: note: called from:
              return make_impl<Tag>::apply(static_cast<X&&>(x)...);
                                          ^
"/opt/boost/1.72.0/include/boost/hana/filter.hpp", line 115: note: called from:
              return hana::make<S>(
                                  ^
"/opt/boost/1.72.0/include/boost/hana/filter.hpp", line 127: note: called from:
              return filter_impl::filter_helper<Indices>(
                                                        ^
"/opt/boost/1.72.0/include/boost/hana/filter.hpp", line 48: note: called from:
          return Filter::apply(static_cast<Xs&&>(xs),
                              ^
          detected during:
            instantiation of function "lambda [](auto, auto, auto)->auto [with <auto-1>=boost::hana::tuple<boost::hana::type<Acts::SurfaceCollector<Acts::MaterialSurface>>, boost::hana::type<Acts::VolumeCollector<Acts::MaterialVolume>>>, <auto-2>=boost::hana::type_detail::is_valid_fun<lambda [](auto)->boost::hana::type<<unnamed>::type::result_type> &&>, <auto-3>=boost::hana::template_t<Acts::detail::result_type_extractor::extractor_impl>]" at line 77
            instantiation of "const auto Acts::detail::type_collector_t [with helper=Acts::detail::result_type_extractor, items=<Acts::SurfaceCollector<Acts::MaterialSurface>, Acts::VolumeCollector<Acts::MaterialVolume>>]" at line 42 of "/home/nwachuch/bld6/acts/Core/include/Acts/Propagator/ActionList.hpp"
            instantiation of type "Acts::ActionList<actors_t...>::result_type<Acts::Propagator<Acts::StraightLineStepper, Acts::Navigator>::result_type_helper<Acts::SingleCurvilinearTrackParameters<Acts::SinglyCharged>, Acts::ActionList<Acts::SurfaceCollector<Acts::MaterialSurface>, Acts::VolumeCollector<Acts::MaterialVolume>>>::this_result_type> [with actors_t=<Acts::SurfaceCollector<Acts::MaterialSurface>, Acts::VolumeCollector<Acts::MaterialVolume>>]" at line 311 of "/home/nwachuch/bld6/acts/Core/include/Acts/Propagator/Propagator.hpp"
            instantiation of class "Acts::Propagator<stepper_t, navigator_t>::result_type_helper<parameters_t, action_list_t> [with stepper_t=Acts::StraightLineStepper, navigator_t=Acts::Navigator, parameters_t=Acts::SingleCurvilinearTrackParameters<Acts::SinglyCharged>, action_list_t=Acts::ActionList<Acts::SurfaceCollector<Acts::MaterialSurface>, Acts::VolumeCollector<Acts::MaterialVolume>>]" at line 217 of "/home/nwachuch/bld6/acts/Core/src/Material/SurfaceMaterialMapper.cpp"

"/home/nwachuch/bld6/acts/Core/src/Material/SurfaceMaterialMapper.cpp", line 218: error: type name is not allowed
    auto mcResult = result.get<MaterialSurfaceCollector::result_type>();
                               ^

"/home/nwachuch/bld6/acts/Core/src/Material/SurfaceMaterialMapper.cpp", line 218: error: expected an expression
    auto mcResult = result.get<MaterialSurfaceCollector::result_type>();
                                                                      ^

"/home/nwachuch/bld6/acts/Core/src/Material/SurfaceMaterialMapper.cpp", line 219: error: type name is not allowed
    auto mvcResult = result.get<MaterialVolumeCollector::result_type>();
                                ^

"/home/nwachuch/bld6/acts/Core/src/Material/SurfaceMaterialMapper.cpp", line 219: error: expected an expression
    auto mvcResult = result.get<MaterialVolumeCollector::result_type>();
                                                                      ^

5 errors detected in the compilation of "/home/nwachuch/bld6/acts/Core/src/Material/SurfaceMaterialMapper.cpp".
gmake[2]: *** [Core/CMakeFiles/ActsCore.dir/src/Material/SurfaceMaterialMapper.cpp.o] Error 2
gmake[1]: *** [Core/CMakeFiles/ActsCore.dir/all] Error 2
gmake: *** [all] Error 2
```

It is noteworthy, that the errors related to line 218 and 219 most likely are the aftermath of the preceeding error regarding the expected constant value.
