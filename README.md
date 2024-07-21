## Hello GitHub

```C++

template<int N>
struct HelloWorld {
    static void print(std::function<void()> lambda) {
        HelloWorld<N-1>::print(lambda);
        lambda();
        std::cout << "Hello" << std::endl;
        lambda();
        std::cout << "GitHub!" << std::endl;
    }
};

template<>
struct HelloWorld<1> {
    static void print(std::function<void()> lambda) {
        lambda();
        std::cout << "Hello" << std::endl;
        lambda();
        std::cout << "GitHub!" << std::endl;
    }
};
```

![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=2876225417&show_icons=true&theme=radical)


![GitHub](https://img.shields.io/github/followers/2876225417?label=Follow&style=social)



