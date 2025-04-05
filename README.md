# -3
class TreeNode:
    """二叉树节点类，采用左孩子右兄弟表示法"""
    def __init__(self, name: str, is_dir: bool = True, size: int = 0):
        self.name = name      # 节点名称
        self.is_dir = is_dir  # 是否为目录（默认True）
        self.size = size      # 文件大小（目录初始为0）
        self.left = None     # 左子节点（第一个子目录/文件）
        self.right = None    # 右兄弟节点

class FileSystem:
    """文件系统管理类"""
    def __init__(self):
        self.root = TreeNode("root", is_dir=True)  # 根目录
    
    def _find_node(self, path: str) -> TreeNode:
        """根据路径查找节点，返回最后一个节点和父节点"""
        if path == "/":
            return self.root
        parts = [p for p in path.split("/") if p]
        current = self.root
        parent = None
        
        for part in parts:
            found = False
            child = current.left  # 第一个子节点
            while child:
                if child.name == part:
                    parent = current
                    current = child
                    found = True
                    break
                child = child.right
            if not found:
                raise FileNotFoundError(f"路径不存在: {path}")
        return current

    def create(self, path: str, is_dir: bool = True, size: int = 0):
        """创建目录/文件"""
        if path == "/":
            raise ValueError("根目录已存在")
        
        parts = [p for p in path.split("/") if p]
        parent_path = "/" + "/".join(parts[:-1])
        name = parts[-1]
        
        try:
            parent_node = self._find_node(parent_path)
        except FileNotFoundError:
            raise FileNotFoundError(f"父路径不存在: {parent_path}")
        
        # 检查是否已存在同名节点
        child = parent_node.left
        while child:
            if child.name == name:
                raise FileExistsError(f"名称已存在: {name}")
            child = child.right
        
        # 创建新节点并插入到左孩子的最右侧兄弟
        new_node = TreeNode(name, is_dir, size)
        if parent_node.left is None:
            parent_node.left = new_node
        else:
            current = parent_node.left
            while current.right:
                current = current.right
            current.right = new_node

    def delete(self, path: str):
        """删除目录/文件（递归删除子树）"""
        node = self._find_node(path)
        parent = self._find_node("/".join(path.split("/")[:-1]))
        
        # 在父节点的子链表中移除当前节点
        prev = None
        current = parent.left
        while current:
            if current == node:
                if prev:
                    prev.right = current.right
                else:
                    parent.left = current.right
                break
            prev = current
            current = current.right
        
        # 递归释放内存（Python自动GC，此处模拟操作）
        def _delete_recursive(node: TreeNode):
            if node.left:
                _delete_recursive(node.left)
            if node.right:
                _delete_recursive(node.right)
            node.left = node.right = None
        
        _delete_recursive(node)

    def rename(self, path: str, new_name: str):
        """重命名节点"""
        node = self._find_node(path)
        node.name = new_name

    def list_files_by_type(self, file_type: str) -> list:
        """中序遍历按文件类型分类"""
        result = []
        
        def _in_order(node: TreeNode):
            if node:
                _in_order(node.left)
                if not node.is_dir and node.name.endswith(file_type):
                    result.append(node.name)
                _in_order(node.right)
        
        _in_order(self.root)
        return result

    def calculate_size(self, path: str = "/") -> int:
        """后序遍历计算目录大小"""
        node = self._find_node(path)
        
        def _post_order(node: TreeNode) -> int:
            if not node:
                return 0
            left_size = _post_order(node.left)
            right_size = _post_order(node.right)
            if node.is_dir:
                node.size = left_size + right_size
            return node.size + (0 if node.is_dir else node.size)
        
        return _post_order(node)

    def visualize(self, node: TreeNode = None, prefix: str = "", is_last: bool = True):
        """可视化目录结构（文本树形输出）"""
        if node is None:
            node = self.root
        
        connectors = {
            "space": "    ",
            "branch": "├── ",
            "last_branch": "└── ",
            "vertical": "│   "
        }
        
        current_prefix = connectors["last_branch"] if is_last else connectors["branch"]
        print(prefix + current_prefix + node.name + 
              f" ({'dir' if node.is_dir else 'file'}, {node.size}KB)")
        
        if node.is_dir:
            children = []
            child = node.left
            while child:
                children.append(child)
                child = child.right
            
            for i, child in enumerate(children):
                new_prefix = prefix + ("    " if is_last else connectors["vertical"])
                self.visualize(child, new_prefix, i == len(children)-1)

# ---------------------- 测试用例 ----------------------
if __name__ == "__main__":
    fs = FileSystem()
    
    # 创建目录结构
    fs.create("/Documents", is_dir=True)
    fs.create("/Downloads", is_dir=True)
    fs.create("/Documents/report.txt", is_dir=False, size=50)
    fs.create("/Documents/image.jpg", is_dir=False, size=70)
    fs.create("/Downloads/movie.mp4", is_dir=False, size=150)
    
    # 可视化测试
    print("\n目录结构：")
    fs.visualize()
    
    # 文件分类测试
    print("\n所有.txt文件：")
    print(fs.list_files_by_type(".txt"))  # ['report.txt']
    
    # 大小统计测试
    print("\n计算根目录大小：")
    print(fs.calculate_size())  # 50+70+150 = 270
    
    # 删除操作测试
    fs.delete("/Downloads/movie.mp4")
    print("\n删除后的目录结构：")
    fs.visualize()
