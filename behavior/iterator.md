# 반복자 패턴

## 이진 트리

```cpp
template<class T>
struct BinaryTree;

template<class T>
struct Node
{
    T value;
    Node<T> * left = nullptr;
    Node<T> * right = nullptr;
    Node<T> * parent = nullptr;
    BinaryTree<T> * tree = nullptr;

    explicit Node(const T & value) :
        value{ value }
    {}

    Node(const T & value, Node<T> * const left, Node<T> * const right) :
        value{ value },
        left{ left },
        right{ right }
    {
        this->left->tree = this->right->tree = tree;
        this->left->parent = this->right->parent = this;
    }

    void set_tree(BinaryTree<T> * t)
    {
        tree = t;
        if (left)
            left->set_tree(t);
        if (right)
            right->set_tree(t);
    }
};

template<class T>
struct BinaryTree
{
    Node<T> * root = nullptr;

    explicit BinaryTree(Node<T> * const root) :
        root{ root }
    {
        root->set_tree(this);
    }
};

template<class T>
struct PreOrderIterator
{
    Node<T> * current;

    explicit PreOrderIterator(Node<T> * current) :
        current(current)
    {}

    bool operator != (const PreOrderIterator<T> * other)
    {
        return current != other.current;
    }

    Node<T> & operator * ()
    {
        return *current;
    }

    PreOrderIterator<T> & operator ++ ()
    {
        if (current->right)
        {
            current = current->right;
            while (current->left)
                current = curent->left;
        }
        else
        {
            Node<T> * p = current->parent;
            while (p && current == p->right)
            {
                current = p;
                p = p->parent;
            }
            current = p;
        }
        return *this;
    }

    PreOrderIterator<T> begin()
    {
        Node<T> * n = root;
        if (n)
            while (n->left)
                n = n->left;
        return PreOrderIterator<T>{ n };
    }

    PreOrderIterator<T> end()
    {
        return PreOrderIterator<T>{ nullptr };
    }
};

template<class T>
struct pre_order_traversal
{
    pre_order_traversal(BinaryTree<T> & tree) :
        tree{ tree }
    {}

    PreOrderIterator<T> begin()
    {
        return tree.begin();
    }

    PreOrderIterator<T> end()
    {
        return tree.end();
    }

private:
    BinaryTree<T> & tree;
};
```

## 코루틴을 이용한 순회

* 구조 예시

```cpp
template<class T>
std::generator<Node<T> *> post_order_impl(Node<T> * node)
{
    if (node)
    {
        for (auto x : post_order_impl(node->left))
            co_yield x;
        for (auto x : post_order_impl(node->right))
            co_yield x;
        co_yield node;
    }
}

template<class T>
std::generator<Node<T> *> post_order(BinaryTree<T> tree)
{
    return post_order_impl(tree.root);
}
```
