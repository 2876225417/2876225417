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

![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=2876225417&layout=compact&theme=dark)

![你的 GitHub wakatime stats](https://github-readme-stats.vercel.app/api/wakatime?username=ppQwQqq)

[![Website](https://img.shields.io/badge/website-ppqwqqq.space-000000?style=flat-square&logo=google-chrome)](https://ppqwqqq.space)

![GitHub](https://img.shields.io/github/followers/2876225417?label=Follow&style=social)




