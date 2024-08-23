## Hello GitHub

```C++
template<class T>
class Stack{
  Stack();
  ~Stack();
};

typedef std::unique_ptr<Stack<int>> p;
typedef std::unique_ptr<std::shared_ptr<Stack<int>>> pp;
typedef std::unique_ptr<std::shared_ptr<std::queue<Stack<int>>>> ppq;
typedef std::unique_ptr<std::shared_ptr<std::queue<std::weak_ptr<Stack<int>>>>> ppqw;
typedef std::unique_ptr<std::shared_ptr<std::queue<std::weak_ptr<std::queue<Stack<int>>>>>> ppqwq;
typedef std::unique_ptr<std::shared_ptr<std::queue<std::weak_ptr<std::queue<std::queue<Stack<int>>>>>>> ppqwqq;
typedef std::unique_ptr<std::shared_ptr<std::queue<std::weak_ptr<std::queue<std::queue<std::queue<Stack<int>>>>>>>> ppqwqqq;

auto ppQwQqq = [](auto&& self, auto&& type) {
    using namespace std;
    if constexpr (is_same_v<std::decay_t<decltype(type)>, ppqwqqq>) cout << "hello github, I'm ppqwqqq";
    else self(self, *type);
};

```

![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=2876225417&show_icons=true&theme=radical)

![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=2876225417&layout=compact&theme=dark)

![你的 GitHub wakatime stats](https://github-readme-stats.vercel.app/api/wakatime?username=ppQwQqq)




