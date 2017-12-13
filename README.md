# async_worker.h

Light C++ wrapper around libuv's `uv_queue_work`.

```cc
class FileStreamAsyncWorker : public AsyncWorker<int, const char> {
  public:
    FileStreamAsyncWorker(
        uv_loop_t* loop,
        const char& input,
      ) : AsyncWorker(loop, input) {}

  private:
    int* onwork(const char& file) {
      int* chunks = count_chunks(file, CHUNK_SIZE);
      return chunks;
    }

    void ondone(int* result, int status) {
      ASSERT(status == 0);
      fprintf(stderr, "read %d chunks\n", *result);
      delete result;
    }
};

int main(int argc, char *argv[]) {
  uv_loop_t* loop = uv_default_loop();
  const char& file = *argv[0];

  FileStreamAsyncWorker worker(loop, file);
  int r = worker.work();
  ASSERT(r == 0);

  uv_run(loop, UV_RUN_DEFAULT);

  uv_loop_close(loop);
}
```

Find more examples at the [thread communication with libuv playground](https://github.com/thlorenz/thread-communication-libuv/tree/master/test).

## API

To use the async worker just extend from the below class and implement its virtual methods, as shown in the above
example.

```cc
class AsyncWorker {
  AsyncWorker(uv_loop_t* loop, U& input);

  // Implemented by inheritor, called on worker thread
  virtual T* onwork(U& input) = 0;

  // Implemented by inheritor,  called on loop thread
  virtual void ondone(T* result, int status) = 0;
}
```

## LICENSE

MIT
