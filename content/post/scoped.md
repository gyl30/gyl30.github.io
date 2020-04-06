---
title: "Scoped"
date: 2020-04-06T20:12:56+08:00
---

<!--more-->

一段有趣的代码

```c++
int run()
{
    auto connection = socket.accept();
    std::string header = connection.read_header();
    if (header.empty())
    {
        connection.close();
        return INVALID_CONNECTION;
    }

    std::string body = connection.read_body();
    if (body.empty())
    {
        connection.close();
        return INVALID_CONNECTION;
    }
    std::string response = process_request(header, body);
    if (response.empty())
    {
        connection.close();
        return INVALID_CONNECTION;
    }
    connection.send(response);
    connection_list.push_back(connection);
    return OK;
}

```



上面的代码每个判断中都需要调用 connection 的 close 方法



利用 RAII 实现一个简单的 scoped_exit

```c++
int run()
{
    auto connection = socket.accept();
    auto clean_up = make_scoped_exit([conn=connection;](){conn.close();});
    std::string header = connection.read_header();
    if (header.empty())
    {
        return INVALID_CONNECTION;
    }

    std::string body = connection.read_body();
    if (body.empty())
    {
        return INVALID_CONNECTION;
    }
    std::string response = process_request(header, body);
    if (response.empty())
    {
        return INVALID_REQUEST;
    }
    connection.send(response);
    connection_list.push_back(connection);
    clean_up.cancel(); // cancel
    return OK;
}

```



通过 make_scoped_exit 注册一个清理函数，clean_up 是一个栈对象，超出作用域后会被自动析构，在析构的时候会调用注册的清理函数。所以下面的错误处理的代码直接 return 即可。正常情况通过 cancel 取消注册的清理函数。



scoped_exit 的实现也很简单

```c++
template<typename Callback>
class scoped_exit {
    public:
    template<typename C>
    scoped_exit( C&& c ):callback( std::forward<C>(c) ){}

    scoped_exit( scoped_exit&& mv )
        :callback( std::move( mv.callback ) ),canceled(mv.canceled)
        {
            mv.canceled = true;
        }

    scoped_exit( const scoped_exit& ) = delete;
    scoped_exit& operator=( const scoped_exit& ) = delete;

    ~scoped_exit()
    {
        if (!canceled)
            try { callback(); } catch( ... ) {}
    }

    scoped_exit& operator = ( scoped_exit&& mv )
    {
        if( this != &mv )
        {
            ~scoped_exit();
            callback = std::move(mv.callback);
            canceled = mv.canceled;
            mv.canceled = true;
        }
        return *this;
    }

    void cancel() { canceled = true; }

    private:
    Callback callback;
    bool canceled = false;
};

template<typename Callback>
scoped_exit<Callback> make_scoped_exit( Callback&& c )
{
    return scoped_exit<Callback>( std::forward<Callback>(c) );
}


```



### 其他

[boost](https://theboostcpplibraries.com/boost.scopeexit)

[stackoverflow](https://stackoverflow.com/questions/3669833/c11-scope-exit-guard-a-good-idea)

[gsl.util](https://github.com/microsoft/GSL/blob/master/include/gsl/gsl_util)

