确保转换器接口使用 `@Mapping(target = "deptName", ignore = true)`注解，避免 MapStruct 尝试自动映射 `deptName`字段。部门名称应在 Service 层手动设置。

 MapStruct 自动映射的是form

```

    /**
     * 实体转VO - 忽略deptName字段，在Service层手动设置
     */
    @Mapping(target = "deptName", ignore = true)
    AioveuPositionVO toVO(AioveuPosition entity);

    /**
     * 实体列表转VO列表 - 忽略deptName字段
     */
    List<AioveuPositionVO> toVO(List<AioveuPosition> entities);
```

```
   /**
     * 实体转VO - 忽略deptName字段，在Service层手动设置
     */
    @Override
    public AioveuPositionVO toVO(AioveuPosition entity) {
        if ( entity == null ) {
            return null;
        }

        AioveuPositionVO aioveuPositionVO = new AioveuPositionVO();

        aioveuPositionVO.setPositionId( entity.getPositionId() );
        aioveuPositionVO.setPositionName( entity.getPositionName() );
        aioveuPositionVO.setDeptId( entity.getDeptId() );
        aioveuPositionVO.setPositionLevel( entity.getPositionLevel() );
        aioveuPositionVO.setDescription( entity.getDescription() );
        aioveuPositionVO.setCreateTime( entity.getCreateTime() );
        aioveuPositionVO.setUpdateTime( entity.getUpdateTime() );

        return aioveuPositionVO;
    }


    /**
     * 实体列表转VO列表 - 忽略deptName字段
     */
    @Override
    public List<AioveuPositionVO> toVO(List<AioveuPosition> entities) {
        if ( entities == null ) {
            return null;
        }

        List<AioveuPositionVO> list = new ArrayList<AioveuPositionVO>( entities.size() );
        for ( AioveuPosition aioveuPosition : entities ) {
            list.add( toVO( aioveuPosition ) );
        }

        return list;
    }
```

