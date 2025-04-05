#include <iostream>
#include <string>
#include <unordered_map>
#include <vector>
using namespace std;

// 树节点结构体
struct TreeNode {
    string name;
    bool isFile;    // true=文件，false=目录
    int size;       // 文件大小（目录为0）
    TreeNode* left;  // 第一个子节点（子目录/文件）
    TreeNode* right; // 兄弟节点
    TreeNode(string n, bool f, int s = 0) : name(n), isFile(f), size(s), left(nullptr), right(nullptr) {}
};

class FileSystem {
private:
    TreeNode* root; // 根目录
    unordered_map<string, vector<TreeNode*>> cache; // 缓存文件分类结果

public:
    FileSystem() {
        root = new TreeNode("Root", false);
    }

    //--- 核心功能：增删改 ---//
    // 创建文件/目录
    TreeNode* createNode(const string& name, bool isFile, int size = 0) {
        return new TreeNode(name, isFile, size);
    }

    // 插入节点（目录或文件）
    bool insertNode(TreeNode* parent, TreeNode* newNode) {
        if (!parent || parent->isFile) {
            cerr << "错误：父节点不是目录！" << endl;
            return false;
        }
        // 检查同名节点
        TreeNode* sibling = parent->left;
        while (sibling) {
            if (sibling->name == newNode->name) {
                cerr << "错误：名称 '" << newNode->name << "' 已存在！" << endl;
                return false;
            }
            sibling = sibling->right;
        }
        // 插入到子节点链表末尾
        if (!parent->left) {
            parent->left = newNode;
        } else {
            sibling = parent->left;
            while (sibling->right) {
                sibling = sibling->right;
            }
            sibling->right = newNode;
        }
        return true;
    }

    // 删除节点（简化版：仅标记为无效）
    bool deleteNode(TreeNode* parent, const string& name) {
        if (!parent || parent->isFile) return false;
        TreeNode* prev = nullptr;
        TreeNode* curr = parent->left;
        while (curr) {
            if (curr->name == name) {
                if (prev) {
                    prev->right = curr->right;
                } else {
                    parent->left = curr->right;
                }
                delete curr;
                return true;
            }
            prev = curr;
            curr = curr->right;
        }
        cerr << "错误：未找到节点 '" << name << "'！" << endl;
        return false;
    }

    //--- 遍历与统计 ---//
    // 中序遍历（按文件类型分类）
    void inOrderTraversal(TreeNode* node) {
        if (!node) return;
        inOrderTraversal(node->left);
        if (!node->isFile) {
            cout << "目录: " << node->name << endl;
        } else {
            string type = getFileType(node->name);
            cout << "文件: " << node->name << " (类型: " << type << ")" << endl;
            cache[type].push_back(node); // 缓存分类结果
        }
        inOrderTraversal(node->right);
    }

    // 后序遍历统计总大小
    int postOrderSize(TreeNode* node) {
        if (!node) return 0;
        int leftSize = postOrderSize(node->left);
        int rightSize = postOrderSize(node->right);
        return node->size + leftSize + rightSize;
    }

    //--- 辅助函数 ---//
    // 获取文件类型（根据扩展名）
    string getFileType(const string& filename) {
        size_t dot = filename.find_last_of(".");
        if (dot == string::npos) return "未知";
        return filename.substr(dot + 1);
    }

    // 打印树结构（缩进可视化）
    void printTree(TreeNode* node, int depth = 0) {
        if (!node) return;
        for (int i = 0; i < depth; i++) cout << "  ";
        cout << (node->isFile ? "📄 " : "📁 ") << node->name;
        if (node->isFile) cout << " (" << node->size << " KB)";
        cout << endl;
        printTree(node->left, depth + 1);
        printTree(node->right, depth);
    }

    //--- 接口函数 ---//
    TreeNode* getRoot() { return root; }
    void clearCache() { cache.clear(); }
};

// 测试用例
int main() {
    FileSystem fs;
    TreeNode* root = fs.getRoot();

    // 创建子目录和文件
    TreeNode* docs = fs.createNode("Documents", false);
    TreeNode* img = fs.createNode("Image.jpg", true, 1024);
    TreeNode* report = fs.createNode("Report.pdf", true, 2048);
    TreeNode* code = fs.createNode("Code.cpp", true, 512);

    // 插入节点
    fs.insertNode(root, docs);
    fs.insertNode(root, img);
    fs.insertNode(root, report);
    fs.insertNode(docs, code);

    // 打印目录结构
    cout << "目录结构：" << endl;
    fs.printTree(root);

    // 中序遍历分类
    cout << "\n文件分类结果：" << endl;
    fs.inOrderTraversal(root);

    // 后序遍历统计大小
    cout << "\n总文件大小: " << fs.postOrderSize(root) << " KB" << endl;

    // 删除文件测试
    fs.deleteNode(root, "Image.jpg");
    cout << "\n删除后的目录结构：" << endl;
   fs.printTree(root);

    return 0;
}
