import java.util.*;

public class EmergencyAmbulanceOptimization {

    // Travel Cost Matrix (Adjacency Matrix)
    static int[][] costMatrix = {
        {0, 15, 25, 35},
        {15, 0, 30, 28},
        {25, 30, 0, 20},
        {35, 28, 20, 0}
    };

    // Location names
    static String[] locations = {
        "Hospital",
        "Emergency Location B",
        "Emergency Location C",
        "Emergency Location D"
    };

    // Validate the input matrix before running an algorithm.
    private static void validateMatrix(int[][] dist) {
        if (dist == null || dist.length == 0) {
            throw new IllegalArgumentException("Cost matrix cannot be empty.");
        }

        int n = dist.length;
        for (int i = 0; i < n; i++) {
            if (dist[i] == null || dist[i].length != n) {
                throw new IllegalArgumentException("Cost matrix must be square.");
            }
            for (int j = 0; j < n; j++) {
                if (dist[i][j] < 0) {
                    throw new IllegalArgumentException(
                        "Travel costs cannot be negative.");
                }
            }
        }
    }

    private static String routeText(List<Integer> route) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < route.size(); i++) {
            if (i > 0) sb.append(" -> ");
            sb.append(locations[route.get(i)]);
        }
        return sb.toString();
    }

    // ============================================
    // Greedy Route Optimization
    // ============================================
    public static String greedyEAROP(int[][] dist) {
        validateMatrix(dist);
        int n = dist.length;
        boolean[] visited = new boolean[n];
        List<Integer> route = new ArrayList<>();

        int current = 0; // Hospital
        int totalCost = 0;
        visited[0] = true;
        route.add(0);

        for (int count = 1; count < n; count++) {
            int nearest = -1;
            int smallestCost = Integer.MAX_VALUE;

            for (int next = 0; next < n; next++) {
                if (!visited[next] && dist[current][next] < smallestCost) {
                    smallestCost = dist[current][next];
                    nearest = next;
                }
            }

            visited[nearest] = true;
            route.add(nearest);
            totalCost += dist[current][nearest];
            current = nearest;
        }

        totalCost += dist[current][0];
        route.add(0);

        return "Greedy Ambulance Route: " + routeText(route)
             + " | Total Cost: " + totalCost;
    }

    // ============================================
    // Dynamic Programming Route Optimization
    // ============================================
    public static String dynamicProgrammingEAROP(int[][] dist) {
        validateMatrix(dist);
        int n = dist.length;

        if (n == 1) {
            return "Dynamic Programming Ambulance Route: Hospital -> Hospital"
                 + " | Total Cost: 0";
        }

        int states = 1 << n;
        int[][] memo = new int[n][states];
        String[][] paths = new String[n][states];

        for (int[] row : memo) Arrays.fill(row, -1);

        int visitedAll = (1 << n) - 1;
        int totalCost = dynamicProgrammingEAROPHelper(
            0, 1, dist, memo, visitedAll, paths);

        String pathIndices = paths[0][1];
        List<Integer> route = new ArrayList<>();
        route.add(0);

        if (pathIndices != null && !pathIndices.isEmpty()) {
            String[] parts = pathIndices.split(",");
            for (String part : parts) {
                if (!part.isEmpty()) route.add(Integer.parseInt(part));
            }
        }
        route.add(0);

        return "Dynamic Programming Ambulance Route: "
             + routeText(route) + " | Total Cost: " + totalCost;
    }

    // DP state = current position + set of already visited locations.
    private static int dynamicProgrammingEAROPHelper(
            int pos, int mask, int[][] dist, int[][] memo,
            int VISITED_ALL, String[][] paths) {

        if (mask == VISITED_ALL) {
            return dist[pos][0];
        }

        if (memo[pos][mask] != -1) {
            return memo[pos][mask];
        }

        int bestCost = Integer.MAX_VALUE;
        String bestPath = "";

        for (int next = 0; next < dist.length; next++) {
            if ((mask & (1 << next)) == 0) {
                int candidate = dist[pos][next]
                    + dynamicProgrammingEAROPHelper(
                        next, mask | (1 << next), dist,
                        memo, VISITED_ALL, paths);

                if (candidate < bestCost) {
                    bestCost = candidate;

                    String suffix = paths[next][mask | (1 << next)];
                    bestPath = next + (suffix == null || suffix.isEmpty()
                            ? "" : "," + suffix);
                }
            }
        }

        memo[pos][mask] = bestCost;
        paths[pos][mask] = bestPath;
        return bestCost;
    }

    // ============================================
    // Backtracking Route Optimization
    // ============================================
    private static int bestBacktrackingCost;
    private static List<Integer> bestBacktrackingRoute;

    public static String backtrackingEAROP(int[][] dist) {
        validateMatrix(dist);
        int n = dist.length;

        bestBacktrackingCost = Integer.MAX_VALUE;
        bestBacktrackingRoute = new ArrayList<>();

        boolean[] visited = new boolean[n];
        visited[0] = true;

        StringBuilder path = new StringBuilder("0");
        earopBacktracking(0, dist, visited, n, 1, 0, path);

        List<Integer> route = new ArrayList<>(bestBacktrackingRoute);
        route.add(0);

        return "Backtracking Ambulance Route: "
             + routeText(route)
             + " | Total Cost: " + bestBacktrackingCost;
    }

    private static int earopBacktracking(
            int pos, int[][] dist, boolean[] visited, int n,
            int count, int cost, StringBuilder path) {

        if (count == n) {
            int totalCost = cost + dist[pos][0];

            if (totalCost < bestBacktrackingCost) {
                bestBacktrackingCost = totalCost;
                bestBacktrackingRoute.clear();

                String[] parts = path.toString().split("->");
                for (String part : parts) {
                    bestBacktrackingRoute.add(Integer.parseInt(part));
                }
            }
            return totalCost;
        }

        int best = Integer.MAX_VALUE;

        for (int next = 0; next < n; next++) {
            if (!visited[next]) {
                visited[next] = true;
                path.append("->").append(next);

                int candidate = earopBacktracking(
                    next, dist, visited, n, count + 1,
                    cost + dist[pos][next], path);

                best = Math.min(best, candidate);

                int last = path.lastIndexOf("->");
                path.delete(last, path.length());
                visited[next] = false;
            }
        }

        return best;
    }

    // ============================================
    // Divide and Conquer Route Optimization
    // ============================================
    private static int bestDivideCost;
    private static List<Integer> bestDivideRoute;

    public static String divideAndConquerEAROP(int[][] dist) {
        validateMatrix(dist);
        int n = dist.length;

        bestDivideCost = Integer.MAX_VALUE;
        bestDivideRoute = new ArrayList<>();

        boolean[] visited = new boolean[n];
        visited[0] = true;

        StringBuilder path = new StringBuilder("0");
        divideAndConquerHelper(0, visited, 0, dist, n, path);

        List<Integer> route = new ArrayList<>(bestDivideRoute);
        route.add(0);

        return "Divide & Conquer Ambulance Route: "
             + routeText(route)
             + " | Total Cost: " + bestDivideCost;
    }

    /*
     * The remaining locations are divided into alternative subproblems:
     * choose one unvisited location as the next destination, recursively
     * solve the remaining locations, and keep the minimum-cost solution.
     */
    private static int divideAndConquerHelper(
            int pos, boolean[] visited, int currentCost,
            int[][] dist, int n, StringBuilder path) {

        if (allVisited(visited)) {
            int totalCost = currentCost + dist[pos][0];

            if (totalCost < bestDivideCost) {
                bestDivideCost = totalCost;
                bestDivideRoute.clear();

                String[] parts = path.toString().split("->");
                for (String part : parts) {
                    bestDivideRoute.add(Integer.parseInt(part));
                }
            }
            return totalCost;
        }

        int best = Integer.MAX_VALUE;

        for (int next = 0; next < n; next++) {
            if (!visited[next]) {
                visited[next] = true;
                path.append("->").append(next);

                int candidate = divideAndConquerHelper(
                    next, visited, currentCost + dist[pos][next],
                    dist, n, path);

                best = Math.min(best, candidate);

                int last = path.lastIndexOf("->");
                path.delete(last, path.length());
                visited[next] = false;
            }
        }

        return best;
    }

    // Check whether all emergency locations have been visited.
    private static boolean allVisited(boolean[] visited) {
        for (boolean value : visited) {
            if (!value) return false;
        }
        return true;
    }

    // ============================================
    // Insertion Sort
    // ============================================
    public static String insertionSort(int[] arr) {
        if (arr == null) {
            throw new IllegalArgumentException("Array cannot be null.");
        }

        for (int i = 1; i < arr.length; i++) {
            int key = arr[i];
            int j = i - 1;

            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }

            arr[j + 1] = key;
        }

        return Arrays.toString(arr);
    }

    // ============================================
    // Binary Search
    // ============================================
    public static String binarySearch(int[] arr, int target) {
        if (arr == null) {
            throw new IllegalArgumentException("Array cannot be null.");
        }

        int left = 0;
        int right = arr.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (arr[mid] == target) {
                return String.valueOf(mid);
            } else if (arr[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }

        return "-1";
    }

    // ============================================
    // Min-Heap
    // ============================================
    static class MinHeap {
        private PriorityQueue<Integer> heap = new PriorityQueue<>();

        public void insert(int value) {
            heap.add(value);
        }

        public int extractMin() {
            if (heap.isEmpty()) {
                throw new NoSuchElementException("Heap is empty.");
            }
            return heap.poll();
        }
    }

    // ============================================
    // Splay Tree
    // ============================================
    static class SplayTree {

        private static class Node {
            int key;
            Node left;
            Node right;

            Node(int key) {
                this.key = key;
            }
        }

        private Node root;

        private Node rotateRight(Node x) {
            Node y = x.left;
            x.left = y.right;
            y.right = x;
            return y;
        }

        private Node rotateLeft(Node x) {
            Node y = x.right;
            x.right = y.left;
            y.left = x;
            return y;
        }

        // Splay the requested key to the root.
        private Node splay(Node root, int key) {
            if (root == null || root.key == key) {
                return root;
            }

            if (key < root.key) {
                if (root.left == null) return root;

                if (key < root.left.key) {
                    root.left.left = splay(root.left.left, key);
                    root = rotateRight(root);
                } else if (key > root.left.key) {
                    root.left.right = splay(root.left.right, key);
                    if (root.left.right != null) {
                        root.left = rotateLeft(root.left);
                    }
                }

                return root.left == null ? root : rotateRight(root);
            } else {
                if (root.right == null) return root;

                if (key > root.right.key) {
                    root.right.right = splay(root.right.right, key);
                    root = rotateLeft(root);
                } else if (key < root.right.key) {
                    root.right.left = splay(root.right.left, key);
                    if (root.right.left != null) {
                        root.right = rotateRight(root.right);
                    }
                }

                return root.right == null ? root : rotateLeft(root);
            }
        }

        public void insert(int value) {
            if (root == null) {
                root = new Node(value);
                return;
            }

            root = splay(root, value);

            if (root.key == value) {
                return; // Avoid duplicate keys.
            }

            Node newNode = new Node(value);

            if (value < root.key) {
                newNode.right = root;
                newNode.left = root.left;
                root.left = null;
            } else {
                newNode.left = root;
                newNode.right = root.right;
                root.right = null;
            }

            root = newNode;
        }

        public boolean search(int value) {
            if (root == null) return false;

            root = splay(root, value);
            return root.key == value;
        }
    }

    // ============================================
    // Driver Method
    // ============================================
    public static void main(String[] args) {

        System.out.println(greedyEAROP(costMatrix));
        System.out.println(dynamicProgrammingEAROP(costMatrix));
        System.out.println(backtrackingEAROP(costMatrix));
        System.out.println(divideAndConquerEAROP(costMatrix));

        // Sorting and Searching
        int[] arr = {8, 3, 5, 1, 9, 2};
        insertionSort(arr);

        System.out.println(
            "Sorted Emergency Response Times: "
            + Arrays.toString(arr)
        );

        System.out.println(
            "Binary Search (Response Time 5 found at index): "
            + binarySearch(arr, 5)
        );

        // Min-Heap Test
        MinHeap heap = new MinHeap();
        heap.insert(10);
        heap.insert(3);
        heap.insert(15);

        System.out.println(
            "Min-Heap Extract Minimum Priority Value: "
            + heap.extractMin()
        );

        // Splay Tree Test
        SplayTree tree = new SplayTree();
        tree.insert(20);
        tree.insert(10);
        tree.insert(30);

        System.out.println(
            "Splay Tree Search (Emergency Case 10 found): "
            + tree.search(10)
        );
    }
}
