# Class-Filter
- - - - - - - - - - - - - -  - - - - - - - - - - - - - - - - - - - - - - - - -
    by: hazurl & Cxx
```cpp
#include <iostream>

#include <tuple>
#include <utility>
#include <type_traits>

struct base {
	int value;

        // Value is transformed in some way and returned.
	auto get() const {
		return value * 2;
	}
};

struct getter : base {
	using tag = struct getter_tag{};
};

struct setter : base {
	using tag = struct setter_tag{};
};

template <class... types>
class request
{
  private:
	std::tuple <types...> t;

	template <class Tag, class Type>
	static decltype(auto) get_element(Type&& s) {
		if constexpr (std::is_same <Tag, getter::tag>::value) {
			return std::tuple(std::forward <Type>(s));
		}

		else {
			return std::tuple <>();
		}
	}

  public:
	request(types&&... args) : t(std::forward <types>(args)...) {
	}

	decltype(auto) get() const && {
		return std::apply([](types... args) {
			return std::tuple_cat(get_element <typename types::tag> (std::move(args.get()))...);
		}, t);
	}
};

int main(int argc, const char* argv[]) {
    auto &&[a, b] = request{
	    getter{2},
	    setter{1}, // This should not be returned.
	    getter{5}
    }.get();

    std::cout << a << '\n'; // Should output 4 .
    std::cout << b << '\n'; // Should output 10.
    return 0;
}
```
first version by hazurl - https://godbolt.org/z/l9qP83 \
second version by Cxx - https://godbolt.org/z/w5rZ0e \
third version by Cxx - https://godbolt.org/z/VKVvHl
