关于+load底层的总结

``` xml

 Loading isn't really thread safe.

typedef void(*load_method_t)(id, SEL);

struct loadable_class {
    Class cls;  // may be nil
    IMP method;
};

struct loadable_category {
    Category cat;  // may be nil
    IMP method;
};

// List of classes that need +load called (pending superclass +load)
// This list always has superclasses first because of the way it is constructed
static struct loadable_class *loadable_classes = nil;
static int loadable_classes_used = 0;
static int loadable_classes_allocated = 0;

// List of categories that need +load called (pending parent class +load)
static struct loadable_category *loadable_categories = nil;
static int loadable_categories_used = 0;
static int loadable_categories_allocated = 0;

提供指定的api来实现
//基于基础类的实现
void add_class_to_loadable_list(Class cls)
//基于类别的实现
void add_category_to_loadable_list(Category cat)


void call_load_methods(void)
{
    static bool loading = NO;
    bool more_categories;
    loadMethodLock.assertLocked();
    // Re-entrant calls do nothing; the outermost call will finish the job.
    if (loading) return;
    loading = YES;
    void *pool = objc_autoreleasePoolPush();
    do {
        // 1. Repeatedly call class +loads until there aren't any more
        while (loadable_classes_used > 0) {
            call_class_loads();
        }
        // 2. Call category +loads ONCE
        more_categories = call_category_loads();
        // 3. Run more +loads if there are classes OR more untried categories
    } while (loadable_classes_used > 0  ||  more_categories);
    objc_autoreleasePoolPop(pool);
    loading = NO;
}
```