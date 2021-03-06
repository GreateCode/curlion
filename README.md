# Curlion

## Introduction

Curlion is a C++ wrapper for libcurl multi socket interface. It hides the complexity of multi socket, provides an easy-to-use interface.

## Requirement

A C++11 compatible compiler is required.

Since multi socket's feature, an event-driven mechanism is also required, such as boost.asio, libevent and so on.

## Usage

### User-implemented mechanism

Before using curlion, an event-driven mechanism must be implemented firstly. Because the mechanism is variable, depending on specific situation, it is impossible to wrap it into curlion. So it is user's responsibility to implement it. 

Curlion provides some interfaces to help simplifying the implementation of event-driven mechanism. They are Timer and SocketWatcher, contain pure virtual methods. Implement these two interface like this:

	class MyTimer : public curlion::Timer {
	public:    
    	void Start(long timeout_ms, const std::function<void()>& callback) override {
    		//...
    	}
    
	    void Stop() override {
	    	//...
	    }
	};
	
	class MySocketWatcher : public curlion::SocketWatcher {
	public:
		void Watch(curl_socket_t socket, Event event, const EventCallback& callback) override {
			//...
		}
    
		void StopWatching(curl_socket_t socket) override {
			//...
		}
	};
	
Besides, another interface SocketFactory can also be implemented if socket's opening and closing need to be taken control. Like this:

	class MySocketFactory : public curlion::SocketFactory {
	public:
		curl_socket_t Open(curlsocktype socket_type, const curl_sockaddr* address) override {
			//...
		}
    
		bool Close(curl_socket_t socket) override {
			//...
		}
	};
	
The code shows that the interfaces are simple, and they may not be difficult to understand. However, a lot of details are omitted. The actual implementation may be complex or not, depending on what mechanism is choosen.

### The rest

After implementing event-driven mechanism, all of the reset are very simple. Here are steps for accomplishing a HTTP request.

Firstly, create a ConnectionManager with instances of Timer, SocketWatcher and SocketFactory:

    auto timer = std::make_shared<MyTimer>();
    auto socket_watcher = std::make_shared<MySocketWatcher>();
    auto socket_factory = std::make_shared<MySocketFactory>();
    
    curlion::ConnectionManager connection_manager(socket_factory, socket_watcher, timer);

Secondly, create a HttpConnection, and configurate its properties:

	auto connection = std::make_shared<curlion::HttpConnection>();
	connection->SetUrl("http://www.google.com");
	connection->SetFinishedCallback([ ](const std::shared_ptr<Connection>& connection) {
	
        if (connection->GetResult() == CURLE_OK) {
            std::cout << connection->GetResponseBody() << std::endl;
        }
        else {
            std::cout << "Connection failed with result: " << connection->GetResult() << std::endl;
        }
    });

Finally, start the connection just with a single method call:

	connection_manager.StartConnection(connection);
	
When connection is finished, the callback set by SetFinishedCallback method will be called. Get results just like the code shows.
	

## Example

Currently, there is an example shows how to use curlion cooperating with boost.asio. More examples will be add in the future.

## Documentation

Curlion contains brief documentation in source files. A HTML version documentation can be generated by doxygen using doc/doxygen script file.

More detail documentation about libcurl can be found at [libcurl's documentation page](http://curl.haxx.se/libcurl/c/).