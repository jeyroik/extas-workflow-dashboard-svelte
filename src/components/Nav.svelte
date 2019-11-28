<script>
	export let segment;
</script>

<nav>
    <div class="nav-wrapper">
        <ul id="nav-mobile" class="left hide-on-med-and-down">
            <li class='{segment === undefined ? "active" : ""}'><a href='.'>Главная</a></li>
            <li class='{segment === "schemas" ? "active" : ""}'><a href='schemas'>Схемы</a></li>
            <li class='{segment === "states" ? "active" : ""}'><a href='states'>Состояния</a></li>
            <li class='{segment === "transitions" ? "active" : ""}'><a href='transitions'>Переходы</a></li>
            <li class='{segment === "entities" ? "active" : ""}'><a href='entities'>Шаблоны сущностей</a></li>
        </ul>
    </div>
</nav>
