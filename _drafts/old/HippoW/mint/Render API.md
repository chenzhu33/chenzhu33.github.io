# Render API


```
/**
 * RenderModule运行在异步线程，充当渲染任务分配与处理的角色
 * 对于前端提交过来的任务，会先转换为不同的Action，并做临时存储，
 * 然后每隔一定时间批量处理所有Action。
 */
public class RenderModule {
    /**
     * 临时存储所有action
     */
    private List<Action> actions = new LinkedList<>();
    
    /**
     * template池
     */
    private Map<TemplateId, Template> templatePool;

    /**
     * 创建完整Logic Tree
     * @param logicTree json格式的logic tree
     */
    public void create(String logicTreeJson) {
        Template template = parse(logicTree); // 解析为一个Template
        templatePool.add(templatePool);
        actions.add(new Action(CREATE, template, template.rootNode));
    }

    /**
     * 更新某个节点
     * @param logicNodeJson json格式的需要修改的节点信息，包含template-id, node-id, 全量属性（如果可以做增量更好）
     */
    public void update(String logicNodeJson) {
        Template template = findTemplate(logicNodeJson); // 找到对应模板
        Node node = findNode(template, logicNodeJson); // 从templatePool中找到对应节点，并将属性更新进去
        actions.add(new Action(UPDATE, template, node));
    }

    /**
     * 删除某个节点
     * @param nodeId 要删除的节点的id
     */
    public void delete(String templateId, String nodeId) {
        Template template = findTemplate(templateId); // 找到对应模板
        Node node = findNode(template, nodeId); // 从当前树上找到对应节点
        actions.add(new Action(DELETE, template, node));
    }

    /**
     * 当前batch完成后调用
     */
    public void batchComplete() {
        // Dispatch all actions
        
        // Batch

        // Layout

        // Render
		
		...
    }
}
```


```
public class Action {
    enum ActionType {CREATE , UPDATE , DELETE}
    ActionType actionType;
    Node node;
    Template template;
}
public class CreateAction extends Action {
    actionType = ActionType.CREATE;
    node = rootNode;
}
public class UpdateAction extends Action {
    actionType = ActionType.UPDATE;
    node = updateNode;
}
public class DeleteAction extends Action {
    actionType = ActionType.DELETE;
    node = deleteNode;
}
```